---
layer: 03_software_engineering
type: engineering
tool: Kubernetes
status: growing
tags: [kubernetes, k8s, containers, orchestration, devops, ml-ops]
created: 2026-03-05
---

# Kubernetes Basics

## Purpose

Kubernetes (K8s) is a container orchestration platform that automates deployment, scaling, and operations of containerised workloads. It provides a declarative API: you describe desired state, and the control plane continuously reconciles actual state toward it. In ML contexts K8s underpins scalable training pipelines, model serving infrastructure, and multi-tenant GPU clusters.

## Architecture

### Control Plane vs Data Plane

```
┌─────────────────────────────────────────────────────────┐
│                     Control Plane                        │
│  ┌──────────────┐  ┌──────────┐  ┌────────────────────┐ │
│  │ kube-apiserver│  │  etcd   │  │ kube-scheduler     │ │
│  │  (REST API)  │  │ (state) │  │ (pod placement)    │ │
│  └──────────────┘  └──────────┘  └────────────────────┘ │
│  ┌──────────────────────────────┐                        │
│  │ kube-controller-manager      │ (deployment, RS, HPA…) │
│  └──────────────────────────────┘                        │
└─────────────────────────────────────────────────────────┘
                          │ API
┌─────────────────────────▼───────────────────────────────┐
│                      Worker Nodes                        │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Node 1                                           │  │
│  │  ┌──────────┐  ┌──────────┐  ┌─────────────────┐ │  │
│  │  │ kubelet  │  │kube-proxy│  │ container runtime│ │  │
│  │  └──────────┘  └──────────┘  │ (containerd)    │ │  │
│  │  ┌──────┐  ┌──────┐          └─────────────────┘ │  │
│  │  │ Pod  │  │ Pod  │                               │  │
│  └──└──────┘──└──────┘───────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Implementation Notes

### Core Object Reference

**Pod** — smallest deployable unit; one or more co-located containers sharing network and storage.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: main
      image: myapp:1.4.2
      ports:
        - containerPort: 8000
      resources:
        requests:
          cpu: "250m"
          memory: "512Mi"
        limits:
          cpu: "1"
          memory: "2Gi"
```

**Deployment** — manages a ReplicaSet; handles rolling updates and rollbacks.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: main
          image: myapp:1.4.2
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 10
            periodSeconds: 15
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
            initialDelaySeconds: 5
            periodSeconds: 5
```

**Service Types**

| Type | Access | Use Case |
|------|--------|----------|
| `ClusterIP` | Internal cluster only | Service-to-service communication |
| `NodePort` | External via node IP + high port | Dev/test without load balancer |
| `LoadBalancer` | External via cloud LB | Production ingress |
| `ExternalName` | DNS alias | Pointing to external service |

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8000
```

**ConfigMap and Secret**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "INFO"
  MAX_WORKERS: "4"
---
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  DATABASE_URL: "postgresql://user:pass@db:5432/mydb"
```

Reference in a pod:
```yaml
envFrom:
  - configMapRef:
      name: app-config
  - secretRef:
      name: db-credentials
```

**PersistentVolume / PersistentVolumeClaim**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: model-storage
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard-ssd
  resources:
    requests:
      storage: 50Gi
```

Mount in pod spec:
```yaml
volumes:
  - name: models
    persistentVolumeClaim:
      claimName: model-storage
containers:
  - volumeMounts:
      - name: models
        mountPath: /models
```

### kubectl Cheatsheet

```bash
# Context / cluster management
kubectl config get-contexts
kubectl config use-context my-cluster

# Apply / delete manifests
kubectl apply -f manifest.yaml
kubectl apply -f ./k8s/                  # whole directory
kubectl delete -f manifest.yaml

# Inspect
kubectl get pods -n production -o wide
kubectl get all -n staging
kubectl describe pod my-pod-abc123
kubectl logs my-pod-abc123 -f --tail=100
kubectl logs -l app=my-app --all-containers

# Exec into container
kubectl exec -it my-pod-abc123 -- /bin/sh

# Port-forward (local debugging)
kubectl port-forward svc/my-app-svc 8080:80

# Scaling
kubectl scale deployment my-app --replicas=5

# Rollout management
kubectl rollout status deployment/my-app
kubectl rollout history deployment/my-app
kubectl rollout undo deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=2

# Resource usage
kubectl top nodes
kubectl top pods -n production --sort-by=memory
```

### Resource Requests and Limits

- **Request**: minimum guaranteed resources. Used by scheduler to find a node.
- **Limit**: hard cap. CPU is throttled at limit; memory limit triggers OOMKill.

Rule of thumb: set `request ≈ P50 usage`, `limit ≈ P99 usage + 20%`. For GPU workloads set `limits.nvidia.com/gpu: 1` — K8s does not share GPUs between pods by default.

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
    nvidia.com/gpu: "1"
  limits:
    cpu: "2"
    memory: "4Gi"
    nvidia.com/gpu: "1"
```

### Liveness and Readiness Probes

- **Liveness**: is the container still alive? Failure → container restart.
- **Readiness**: is the container ready to serve traffic? Failure → removed from Service endpoints.
- **Startup**: is the container done initialising? Disables liveness checks until it passes (useful for slow-starting apps).

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /readyz
    port: 8080
  periodSeconds: 5
  failureThreshold: 2

startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10              # allows 5 minutes for startup
```

### Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
    - type: Resource
      resource:
        name: memory
        target:
          type: AverageValue
          averageValue: 1.5Gi
```

Custom metrics (e.g. request queue depth) require an adapter like `kube-state-metrics` + Prometheus Adapter.

### Ingress Controllers

Ingress exposes HTTP/S routes from outside the cluster to Services:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - api.example.com
      secretName: api-tls
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-svc
                port:
                  number: 80
```

Popular controllers: **NGINX Ingress**, **Traefik**, **AWS ALB Ingress**, **GKE Ingress**.

### Helm Charts

Helm is the de-facto package manager for K8s. A chart is a directory of templated YAML.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-postgres bitnami/postgresql --set auth.postgresPassword=secret

# Override values
helm install my-app ./chart -f values.production.yaml

# Upgrade
helm upgrade my-app ./chart --atomic --timeout 5m

# Diff before upgrade (helm-diff plugin)
helm diff upgrade my-app ./chart -f values.production.yaml
```

A minimal chart structure:
```
my-chart/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
    ingress.yaml
    _helpers.tpl
```

### ML Workloads on Kubernetes

**GPU Node Pools**

Nodes with GPUs should be tainted so only GPU workloads are scheduled on them:

```yaml
# Node taint (set by node pool config or manually)
kubectl taint nodes gpu-node-1 nvidia.com/gpu=present:NoSchedule

# Pod toleration
tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
nodeSelector:
  cloud.google.com/gke-accelerator: nvidia-tesla-a100
```

**Batch Training Jobs**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: train-resnet50-run42
spec:
  backoffLimit: 2
  completions: 1
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: trainer
          image: mytrainer:git-a3f8c12
          command: ["python", "train.py", "--config", "configs/resnet50.yaml"]
          resources:
            limits:
              nvidia.com/gpu: "4"
          volumeMounts:
            - name: datasets
              mountPath: /datasets
              readOnly: true
            - name: checkpoints
              mountPath: /checkpoints
      volumes:
        - name: datasets
          persistentVolumeClaim:
            claimName: imagenet-pvc
        - name: checkpoints
          persistentVolumeClaim:
            claimName: checkpoints-pvc
```

**Model Serving — Deployment + Service**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: model-server
  template:
    metadata:
      labels:
        app: model-server
    spec:
      containers:
        - name: triton
          image: nvcr.io/nvidia/tritonserver:24.01-py3
          args: ["tritonserver", "--model-repository=/models"]
          ports:
            - containerPort: 8000  # HTTP
            - containerPort: 8001  # gRPC
          resources:
            limits:
              nvidia.com/gpu: "1"
          readinessProbe:
            httpGet:
              path: /v2/health/ready
              port: 8000
            periodSeconds: 10
```

## Trade-offs

| Pattern | Pro | Con |
|---------|-----|-----|
| Deployment for inference | Auto-restart, rolling update | GPU may not share well — use 1 GPU/pod |
| Job for training | Isolated, retry logic | No live monitoring without extra tooling |
| HPA on CPU | Simple, built-in | Lags on burst traffic; need warm replicas |
| Helm | Reusable, versioned | Template complexity; easy to drift from source |
| Shared GPU (MIG) | Cost efficient | Requires A100/H100, complex setup |

## References

- [Kubernetes official docs](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/overview.html)
- [Helm docs](https://helm.sh/docs/)
- [KubeFlow — ML Pipelines on K8s](https://www.kubeflow.org/)

## Links
- [[docker_patterns|Docker]]
- [[cicd_pipelines|CI/CD]]
- [[04_ml_engineering/08_infrastructure_and_platform/ml_environment_management|ML Environment Management]]
