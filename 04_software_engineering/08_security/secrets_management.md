---
layer: 04_software_engineering
type: pattern
status: growing
tags: [pattern, safety, deployment]
created: 2026-03-07
---

# Secrets Management

## The Problem

A **secret** is any value that grants access: passwords, API keys, JWT signing keys, TLS private keys, database credentials, OAuth client secrets. Secrets hardcoded in source code or committed to a repository are exposed to everyone with read access to that repo — forever, because git history persists even after deletion.

**Concrete failure modes:**
- Secret committed to a public GitHub repo: scraped by bots within seconds; abuse begins in minutes.
- Secret in a Docker image layer: extractable with `docker history` by anyone who runs the image.
- Secret in application logs: accessible to everyone with log access; often retained for 90+ days.
- Secret shared across environments: a dev-environment breach reaches production.

The 12-factor app principle VI states: *"Config is everything that is likely to vary between deploys (staging, production, developer environments). Apps should store config in environment variables."* The corollary is that secrets must never be in code — they must be injected at runtime.

---

## Environment Variables Pattern

The simplest secure pattern: inject secrets into the process environment at startup, read with `os.environ`.

```python
import os

DATABASE_URL = os.environ["DATABASE_URL"]          # raises KeyError if unset
API_KEY      = os.environ.get("API_KEY", "")       # returns empty string if unset
```

**Why `os.environ["KEY"]` over `os.getenv("KEY")`:** explicit `KeyError` on missing values prevents silent failures where an empty string is used as a credential.

**What the environment variable pattern does NOT protect against:**
- Process memory dumps (the value is in RAM).
- `ps aux` showing env vars if injected via command flags instead of the environment.
- Child processes that inherit the environment.

For most web services, these risks are acceptable. For high-security contexts, use a secrets manager with dynamic short-lived credentials.

---

## Environment Files (.env)

`.env` files are local developer convenience — they collect environment variables for a single project without polluting the shell.

```bash
# .env — never commit this file
DATABASE_URL=postgresql://user:password@localhost/mydb
OPENAI_API_KEY=sk-...
JWT_SECRET=supersecretkey
```

Load with [`python-dotenv`](https://pypi.org/project/python-dotenv/):

```python
from dotenv import load_dotenv
load_dotenv()  # reads .env into os.environ; no-op if the file is absent

import os
db_url = os.environ["DATABASE_URL"]
```

**Critical `.gitignore` rules:**

```gitignore
# .gitignore
.env
.env.*
!.env.example   # allow a template with placeholder values
```

Always commit a `.env.example` file with placeholder values to document required variables without exposing secrets:

```bash
# .env.example — commit this
DATABASE_URL=postgresql://user:password@host/dbname
OPENAI_API_KEY=<your-key-here>
JWT_SECRET=<generate-with: openssl rand -hex 32>
```

---

## Secret Management in CI/CD

### GitHub Actions

Store secrets in **Settings → Secrets and variables → Actions**. Reference them in workflows as `${{ secrets.SECRET_NAME }}`. GitHub Actions automatically masks secret values in log output.

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY:      ${{ secrets.API_KEY }}
        run: python deploy.py
```

**Secret scanning:** GitHub natively scans all pushes for known secret patterns (AWS keys, GitHub tokens, Stripe keys, etc.) via [Secret Scanning](https://docs.github.com/en/code-security/secret-scanning). Enable it under Security settings. For additional patterns, use [gitleaks](https://github.com/gitleaks/gitleaks) as a pre-commit hook or CI step:

```yaml
- name: Scan for secrets
  uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Pre-commit hook (local prevention)

```bash
pip install detect-secrets
detect-secrets scan > .secrets.baseline
# Add to .pre-commit-config.yaml:
# - repo: https://github.com/Yelp/detect-secrets
#   rev: v1.4.0
#   hooks:
#     - id: detect-secrets
#       args: ['--baseline', '.secrets.baseline']
```

---

## Production Secret Stores

Environment variables injected at container startup suffice for simple deployments. For production systems with multiple services, dynamic credentials, and audit requirements, use a dedicated secret store.

### HashiCorp Vault

Vault is a secrets broker: it stores, encrypts, and dynamically generates credentials. Key capabilities:

- **Dynamic secrets:** Vault generates short-lived database credentials on demand. The credential is never static; it expires after a TTL (e.g., 1 hour).
- **Lease + renewal:** clients hold a lease and must renew it or the secret is revoked.
- **Audit log:** every secret access is logged with identity, timestamp, and path.
- **Transit encryption:** Vault performs encryption as a service — applications send data, Vault returns ciphertext, without the app ever holding the key.

```python
import hvac

client = hvac.Client(url="https://vault.example.com", token=VAULT_TOKEN)

# Read a static secret
secret = client.secrets.kv.v2.read_secret_version(
    path="myapp/database",
    mount_point="secret",
)
db_password = secret["data"]["data"]["password"]

# Dynamic DB credentials (Vault database secrets engine)
creds = client.secrets.database.generate_credentials(name="myapp-role")
username = creds["data"]["username"]
password = creds["data"]["password"]
```

### AWS Secrets Manager

AWS-native alternative. Stores JSON blobs; integrates with IAM for access control.

```python
import boto3, json

client = boto3.client("secretsmanager", region_name="eu-west-1")
response = client.get_secret_value(SecretId="prod/myapp/database")
secret = json.loads(response["SecretString"])
db_password = secret["password"]
```

IAM policy controls which roles/services can access which secrets — no static Vault token required in AWS environments. Secrets Manager natively supports automatic rotation via Lambda.

### Comparison

| | .env | GitHub Actions Secrets | AWS Secrets Manager | HashiCorp Vault |
|---|---|---|---|---|
| **Use case** | Local dev | CI/CD pipelines | AWS-hosted production | Multi-cloud / on-prem |
| **Dynamic credentials** | No | No | Yes (via rotation) | Yes (native) |
| **Audit log** | No | Basic | Yes (CloudTrail) | Yes (fine-grained) |
| **Complexity** | Minimal | Low | Medium | High |
| **Cost** | Free | Free (in limits) | $0.40/secret/month | Self-hosted or HCP |

---

## Rotation Patterns

Static long-lived secrets are a liability. Rotation limits the window of exposure for a compromised secret.

**Rotation strategies:**

1. **Scheduled rotation:** a Lambda/Cloud Function regenerates the secret on a schedule (e.g., every 30 days), updates the store, and notifies dependent services. AWS Secrets Manager and GCP Secret Manager support this natively.

2. **Dual-active rotation:** the old and new secrets are valid simultaneously for a short overlap window. This avoids downtime during rotation when multiple replicas reload at different times.

3. **Dynamic / just-in-time credentials:** Vault's preferred pattern. No rotation needed because each credential has a short TTL (minutes to hours). After expiry, the credential ceases to function; no rotation event required.

**Rotation hygiene:**
- All secrets must have a documented owner and rotation policy (even if "manual, annually").
- Rotation must be tested before expiry, not triggered by an incident.
- After a suspected breach, rotate immediately and audit recent access logs.

---

## References

- [12-Factor App — Config](https://12factor.net/config)
- [HashiCorp Vault documentation](https://developer.hashicorp.com/vault/docs)
- [AWS Secrets Manager documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
- [GitHub Actions — encrypted secrets](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
- [OWASP — Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [gitleaks](https://github.com/gitleaks/gitleaks) — secret scanning tool

---

## Links

- [[04_software_engineering/08_security/index|Security]]
- [[04_software_engineering/07_devops_and_infrastructure/index|DevOps & Infrastructure]]
- [[auth_and_authorization|Authentication and Authorization]] — secrets are used to sign and verify auth tokens
- [[filesystem_sandboxing|Filesystem Sandboxing]] — related secure boundary enforcement
