---
layer: 08_implementations
type: application
status: growing
tags: [pattern, deployment]
created: 2026-03-06
---

# Kubernetes Deployment Patterns

## Purpose

Practical Kubernetes manifests and Helm chart patterns for deploying containerised applications and ML services. Covers Deployment, Service, Ingress, ConfigMap/Secret, HPA, and rolling update strategies. Synthesized from: [[kubernetes_basics|Kubernetes Basics]], [[docker_patterns|Docker Patterns]].

### Examples

**Complete Deployment + Service + Ingress stack:**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inference-api
  labels:
    app: inference-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: inference-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # allow 1 extra pod during update
      maxUnavailable: 0    # never take a pod down before a new one is ready
  template:
    metadata:
      labels:
        app: inference-api
    spec:
      containers:
        - name: inference-api
          image: ghcr.io/myorg/inference-api:1.2.3
          ports:
            - containerPort: 8000
          env:
            - name: MODEL_VERSION
              value: "v3"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
          resources:
            requests:
              cpu: "500m"
              memory: "512Mi"
            limits:
              cpu: "2"
              memory: "2Gi"
          readinessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 30
            periodSeconds: 15

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: inference-api
spec:
  selector:
    app: inference-api
  ports:
    - port: 80
      targetPort: 8000
  type: ClusterIP   # internal only; use Ingress for external access

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: inference-api
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: inference-api
                port:
                  number: 80
  tls:
    - hosts: [api.example.com]
      secretName: api-tls-secret
```

**ConfigMap and Secret:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  FEATURE_X: "enabled"

---
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:          # stringData auto-base64-encodes
  password: "supersecret"
```

**Horizontal Pod Autoscaler (CPU-based):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: inference-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: inference-api
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**Helm chart structure:**
```
charts/inference-api/
├── Chart.yaml          # name, version, appVersion
├── values.yaml         # default values
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   └── _helpers.tpl    # template helpers
└── values-prod.yaml    # override for production
```

**`values.yaml`:**
```yaml
replicaCount: 2
image:
  repository: ghcr.io/myorg/inference-api
  tag: latest
  pullPolicy: IfNotPresent
service:
  port: 80
ingress:
  enabled: true
  host: api.example.com
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 2
    memory: 2Gi
hpa:
  enabled: true
  minReplicas: 2
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
```

**Deploy with Helm:**
```bash
# Install
helm install inference-api ./charts/inference-api \
  --namespace prod --create-namespace \
  -f charts/inference-api/values-prod.yaml

# Upgrade
helm upgrade inference-api ./charts/inference-api \
  --namespace prod -f charts/inference-api/values-prod.yaml

# Rollback
helm rollback inference-api 1 --namespace prod
```

**kubectl essentials:**
```bash
# Check rollout status
kubectl rollout status deployment/inference-api

# View logs
kubectl logs -l app=inference-api --tail=100 -f

# Exec into pod
kubectl exec -it deploy/inference-api -- bash

# Port-forward for local testing
kubectl port-forward svc/inference-api 8080:80
```

## Architecture

```
External traffic
    │  HTTPS
    ▼
Ingress (nginx) → TLS termination
    │
    ▼
Service (ClusterIP) → round-robin load balancing
    │
    ├── Pod 1 (inference-api)
    ├── Pod 2 (inference-api)   ← HPA adds/removes pods based on CPU
    └── Pod 3 (inference-api)
         │
         ▼
    ConfigMap (env config) + Secret (credentials)
```

**Rolling update sequence:**
1. HPA or `kubectl set image` triggers new ReplicaSet
2. K8s creates new pod (maxSurge); waits for readiness probe to pass
3. Old pod is terminated (maxUnavailable=0 means zero-downtime)
4. Process repeats until all replicas are on new version

## Links
- [[kubernetes_basics|Kubernetes Basics]]
- [[docker_patterns|Docker Patterns]]
- [[cicd_pipelines|CI/CD Pipelines]]
- [[github_actions|GitHub Actions]]
- [[05_ml_engineering/09_infrastructure_and_platform/ml_environment_management|ML Environment Management]]
