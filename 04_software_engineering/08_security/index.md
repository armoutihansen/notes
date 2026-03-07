---
layer: 04_software_engineering
type: index
status: evergreen
tags: []
created: 2026-03-05
---

# Security

Defensive engineering patterns for secure-by-default software: filesystem sandboxing, authentication, and trust boundary enforcement.

← Prev [[04_software_engineering/07_devops_and_infrastructure/index|← DevOps & Infrastructure]] | Next → [[04_software_engineering/09_ai_assisted_software_engineering/index|AI-Assisted Engineering →]]

## Notes

- [[filesystem_sandboxing|Filesystem Sandboxing]] — path traversal prevention, `os.path.commonpath`, chroot, and container isolation for LLM tool functions
- [[auth_and_authorization|Authentication and Authorization]] — JWT validation, OAuth2 flows (client credentials, auth code + PKCE), API keys, RBAC, IDOR, token leakage
- [[secrets_management|Secrets Management]] — .env patterns, CI/CD secret injection, HashiCorp Vault, AWS Secrets Manager, rotation strategies
- [[github_permissions|GitHub Permissions]] — fine-grained PATs, GITHUB_TOKEN scopes, org/repo roles, environments and deployment gates

## Links
- [[04_software_engineering/index|Software Engineering]]
- [[04_software_engineering/07_devops_and_infrastructure/index|DevOps & Infrastructure]]
- [[04_software_engineering/09_ai_assisted_software_engineering/index|AI-Assisted Engineering]]
- [[04_software_engineering/09_ai_assisted_software_engineering/mcp_protocol|MCP Protocol]]
