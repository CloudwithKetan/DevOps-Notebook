# Kubernetes Interview Questions & Answers
> Covers Basics · Architecture · Workloads · Networking · Storage · Security · Scenario-Based

---

## 🟢 SECTION 1: BASICS & CORE CONCEPTS

---

### 1. What is Kubernetes and why is it used?
**Answer:**
Kubernetes (K8s) is an open-source container orchestration platform originally developed by Google. It automates the deployment, scaling, load balancing, and management of containerized applications.

**Why it's used:**
- Automates container lifecycle (start, stop, restart, reschedule)
- Self-healing — restarts failed containers, replaces unhealthy nodes
- Horizontal scaling — scale up/down based on load
- Service discovery and load balancing built-in
- Declarative configuration via YAML manifests
- Works across on-prem, cloud, and hybrid environments

---

### 2. What are the main components of a Kubernetes cluster?
**Answer:**

**Control Plane (Master Node):**
| Component | Role |
|---|---|
| `kube-apiserver` | Front door of the cluster; all communication goes through it |
| `etcd` | Distributed key-value store; source of truth for cluster state |
| `kube-scheduler` | Assigns pods to nodes based on resources and constraints |
| `kube-controller-manager` | Runs controllers (node, replication, endpoints, etc.) |
| `cloud-controller-manager` | Integrates with cloud provider APIs |

**Worker Nodes:**
| Component | Role |
|---|---|
| `kubelet` | Agent on each node; ensures containers in pods are running |
| `kube-proxy` | Manages network rules for pod communication |
| Container Runtime | Runs containers (containerd, CRI-O, Docker) |

---

### 3. What is a Pod?
**Answer:**
A Pod is the smallest and most basic deployable unit in Kubernetes. It represents one or more containers that:
- Share the same network namespace (same IP address and ports)
- Share the same storage volumes
- Are always co-located and co-scheduled on the same node

Most pods run a single container, but multi-container pods are used for sidecar patterns (e.g., logging agent alongside the main app).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-app
    image: nginx:1.21
    ports:
    - containerPort: 80
```

---

### 4. What is the difference between a Pod, Deployment, and ReplicaSet?
**Answer:**

| Resource | Purpose |
|---|---|
| **Pod** | Basic unit; runs one or more containers. No self-healing. |
| **ReplicaSet** | Ensures a specified number of pod replicas are running. Replaces failed pods. |
| **Deployment** | Manages ReplicaSets. Adds rolling updates, rollbacks, and version history. |

**In practice:** You almost never create Pods or ReplicaSets directly. You create a Deployment, which manages a ReplicaSet, which manages Pods.

```
Deployment → manages → ReplicaSet → manages → Pods
```

---

### 5. What is etcd and why is it critical?
**Answer:**
`etcd` is a distributed, consistent key-value store that serves as Kubernetes' database — it stores the entire cluster state (all resource definitions, configurations, secrets, etc.).

**Why critical:**
- If etcd is lost without a backup, the entire cluster state is gone
- All API server reads/writes go through etcd
- etcd uses the Raft consensus algorithm for high availability
- Must be backed up regularly in production

```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
```

---

### 6. What is a Namespace?
**Answer:**
Namespaces provide a mechanism for isolating groups of resources within a single cluster. They're like virtual clusters within a physical cluster.

**Use cases:**
- Separate environments (dev, staging, prod) in one cluster
- Multi-team isolation
- Resource quota enforcement per team

**Default namespaces:**
- `default` — Where resources go if no namespace is specified
- `kube-system` — Kubernetes system components
- `kube-public` — Readable by all users; used for cluster info
- `kube-node-lease` — Node heartbeat objects

```bash
kubectl create namespace dev-team
kubectl get pods -n dev-team
kubectl get pods --all-namespaces
```

---

### 7. What is `kubectl` and what are the most common commands?
**Answer:**
`kubectl` is the command-line tool for interacting with a Kubernetes cluster via the API server.

```bash
kubectl get pods                          # List pods
kubectl get pods -o wide                  # With node info
kubectl describe pod <pod-name>           # Detailed info + events
kubectl logs <pod-name>                   # View logs
kubectl logs <pod-name> -c <container>    # Logs from specific container
kubectl exec -it <pod-name> -- bash       # Shell into pod
kubectl apply -f manifest.yaml            # Apply config
kubectl delete -f manifest.yaml           # Delete resources
kubectl get all -n <namespace>            # All resources in namespace
kubectl top pods                          # CPU/memory usage
kubectl rollout status deployment/<name>  # Check rollout
kubectl scale deployment <name> --replicas=5
```

---

### 8. What is a Node in Kubernetes?
**Answer:**
A Node is a worker machine (physical or virtual) in the Kubernetes cluster. Each node runs pods and is managed by the control plane. Every node runs:
- `kubelet` — communicates with the API server
- `kube-proxy` — handles network routing
- Container runtime — actually runs containers

```bash
kubectl get nodes              # List all nodes
kubectl describe node <name>   # Node details, capacity, conditions
kubectl cordon <node>          # Mark node unschedulable
kubectl drain <node>           # Evict all pods, then cordon
kubectl uncordon <node>        # Make node schedulable again
```

---

### 9. What is the role of `kube-scheduler`?
**Answer:**
The scheduler watches for newly created Pods with no assigned node and selects the best node for them based on:
- Resource requirements — CPU, memory requests/limits
- Node affinity/anti-affinity rules
- Taints and tolerations
- Pod affinity/anti-affinity
- Node capacity — available resources

The scheduling happens in two phases:
1. **Filtering** — Eliminate nodes that don't meet requirements
2. **Scoring** — Rank remaining nodes; pick highest score

---

### 10. What is `kubelet`?
**Answer:**
`kubelet` is a node agent that runs on every worker node. It:
- Registers the node with the API server
- Watches the API server for pod assignments
- Starts/stops containers via the container runtime
- Reports node and pod status back to the control plane
- Performs health checks (liveness/readiness probes)
- Mounts volumes to pods

---

## 🔵 SECTION 2: WORKLOADS & CONTROLLERS

---

### 11. What is a Deployment and how does rolling update work?
**Answer:**
A Deployment manages the desired state of a set of pods. Rolling updates allow you to update pods gradually without downtime.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Max extra pods during update
      maxUnavailable: 1    # Max pods that can be down
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:v2
```

```bash
kubectl rollout status deployment/my-app    # Watch rollout
kubectl rollout history deployment/my-app   # Version history
kubectl rollout undo deployment/my-app      # Rollback to previous
kubectl rollout undo deployment/my-app --to-revision=2
```

---

### 12. What is a StatefulSet and when do you use it?
**Answer:**
StatefulSets manage stateful applications where each pod needs a stable identity and persistent storage.

**Key differences from Deployments:**
- Pods have stable, predictable names: `pod-0`, `pod-1`, `pod-2`
- Pods are created/deleted in order (not simultaneously)
- Each pod gets its own PersistentVolumeClaim
- Stable network identity via Headless Service

**Use cases:** Databases (MySQL, PostgreSQL, MongoDB, Cassandra), Kafka, ZooKeeper, etcd.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 3
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

---

### 13. What is a DaemonSet?
**Answer:**
A DaemonSet ensures that exactly one Pod runs on every (or selected) node in the cluster. New nodes automatically get the pod; removed nodes get the pod garbage collected.

**Use cases:**
- Log collectors (Fluentd, Filebeat)
- Monitoring agents (Prometheus Node Exporter, Datadog)
- Network plugins (Calico, Weave)
- Security agents

---

### 14. What is a Job and CronJob?
**Answer:**

**Job** — Runs one or more pods to completion. Useful for batch tasks, migrations, or one-time scripts.

**CronJob** — Runs a Job on a schedule (like Linux cron).

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 2 * * *"   # Every day at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
            command: ["/bin/sh", "-c", "pg_dump mydb > /backup/dump.sql"]
          restartPolicy: OnFailure
```

---

### 15. What are Init Containers?
**Answer:**
Init containers run to completion before the main application containers start. They run sequentially — if any fails, the pod restarts.

**Use cases:**
- Wait for a dependency service to be ready
- Clone a git repo before the app starts
- Database migration before app launch
- Generate config files from templates

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db-service 5432; do sleep 2; done']
  containers:
  - name: my-app
    image: my-app:latest
```

---

### 16. What is a Sidecar container pattern?
**Answer:**
A sidecar is a secondary container in the same pod that extends or enhances the main container without changing it. Both share the same network, volumes, and lifecycle.

**Common sidecars:**
- Logging — Fluentd sidecar collects and ships logs
- Proxy — Envoy sidecar (used in Istio service mesh)
- Sync — Git sync container keeps files up to date
- Monitoring — Prometheus exporter alongside the app

---

### 17. What are Resource Requests and Limits?
**Answer:**

- **Requests** — Minimum resources a container needs. Used by scheduler for placement decisions.
- **Limits** — Maximum resources a container can use. CPU throttled, Memory OOMKilled if exceeded.

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"      # 250 millicores = 0.25 CPU
  limits:
    memory: "256Mi"
    cpu: "500m"
```

**QoS Classes:**
- **Guaranteed** — Requests == Limits (highest priority, last evicted)
- **Burstable** — Requests < Limits
- **BestEffort** — No requests or limits set (first evicted under pressure)

---

### 18. What is a HorizontalPodAutoscaler (HPA)?
**Answer:**
HPA automatically scales pod replica count based on CPU/memory utilization or custom metrics.

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
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

Requires the **Metrics Server** to be installed in the cluster.

---

### 19. What is a VerticalPodAutoscaler (VPA)?
**Answer:**
VPA automatically adjusts CPU and memory requests for containers based on historical usage — unlike HPA which scales replicas, VPA scales resource allocation per pod.

**Modes:** `Off` (recommendations only), `Initial` (set at pod creation), `Auto` (evict and recreate pods)

**Note:** HPA and VPA should not be used together on the same resource type.

---

### 20. What is a LimitRange and ResourceQuota?
**Answer:**

**LimitRange** — Sets default, min, and max resource limits per container/pod within a namespace.

**ResourceQuota** — Limits the total resource consumption across all pods in a namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: dev-team
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
```

---

## 🟡 SECTION 3: NETWORKING

---

### 21. What are the types of Kubernetes Services?
**Answer:**

| Type | Description | Use Case |
|---|---|---|
| **ClusterIP** | Default. Internal cluster IP only. | Internal service communication |
| **NodePort** | Exposes on each Node's IP at a static port. | Dev/testing external access |
| **LoadBalancer** | Creates an external cloud load balancer. | Production external traffic |
| **ExternalName** | Maps service to a DNS name. | Route to external services |

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80         # Service port
    targetPort: 8080 # Container port
```

---

### 22. What is an Ingress and Ingress Controller?
**Answer:**

**Ingress** — A Kubernetes resource that manages external HTTP/HTTPS access with routing rules (host-based, path-based).

**Ingress Controller** — The actual component that reads Ingress rules and implements them (nginx, Traefik, AWS ALB, HAProxy).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
  tls:
  - hosts:
    - api.myapp.com
    secretName: tls-secret
```

---

### 23. What is a Headless Service?
**Answer:**
A Headless Service has `clusterIP: None`. DNS returns the IP addresses of all individual pods instead of a single virtual IP. Essential for StatefulSets where each pod needs direct addressability.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

Pod DNS: `pod-0.mysql-headless.default.svc.cluster.local`

---

### 24. What is a NetworkPolicy?
**Answer:**
NetworkPolicy controls traffic between pods and external endpoints. By default, all pods can communicate freely. NetworkPolicies implement allow/deny rules.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}     # All pods in namespace
  policyTypes:
  - Ingress
  ingress: []         # No rules = deny all ingress
```

**Note:** Requires a CNI plugin that supports NetworkPolicy (Calico, Cilium). Flannel does NOT support it.

---

### 25. How does DNS work in Kubernetes?
**Answer:**
Kubernetes includes CoreDNS that automatically assigns DNS names to Services and Pods.

**Service DNS format:**
```
<service-name>.<namespace>.svc.cluster.local
```

**Examples:**
```bash
# Within same namespace
curl http://my-service

# Cross-namespace
curl http://my-service.other-namespace.svc.cluster.local
```

---

## 🟠 SECTION 4: STORAGE

---

### 26. What is a PersistentVolume (PV) and PersistentVolumeClaim (PVC)?
**Answer:**

**PV** — A piece of storage provisioned by an admin. Cluster-level resource, independent of pod lifecycle.

**PVC** — A request for storage by a user. Kubernetes binds a PVC to a matching PV.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

**Access Modes:**
- `ReadWriteOnce (RWO)` — One node can read/write
- `ReadOnlyMany (ROX)` — Many nodes can read
- `ReadWriteMany (RWX)` — Many nodes can read/write

---

### 27. What is a StorageClass?
**Answer:**
StorageClass defines types of storage and enables dynamic provisioning — PVs are automatically created when a PVC is submitted.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

---

### 28. What is a ConfigMap and Secret?
**Answer:**

**ConfigMap** — Stores non-sensitive configuration data as key-value pairs. Decouples config from container images.

**Secret** — Stores sensitive data (passwords, tokens) base64-encoded. Should be encrypted at rest in production.

```yaml
# Consuming in pods
envFrom:
- configMapRef:
    name: app-config
- secretRef:
    name: db-secret
```

Mounted as files: ConfigMap values auto-update in ~1 minute. Environment variable secrets require pod restart.

---

## 🔴 SECTION 5: SECURITY

---

### 29. What is RBAC in Kubernetes?
**Answer:**
Role-Based Access Control controls who can do what in the cluster.

| Resource | Scope | Purpose |
|---|---|---|
| **Role** | Namespace | Permissions within a namespace |
| **ClusterRole** | Cluster-wide | Permissions across the whole cluster |
| **RoleBinding** | Namespace | Binds a Role to a user/SA |
| **ClusterRoleBinding** | Cluster-wide | Binds a ClusterRole cluster-wide |

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

---

### 30. What is a ServiceAccount?
**Answer:**
A ServiceAccount provides an identity for processes running inside pods to authenticate against the Kubernetes API server. Every namespace has a `default` ServiceAccount.

Best practice: Create dedicated ServiceAccounts per application with minimal permissions (least privilege).

---

### 31. What are Taints and Tolerations?
**Answer:**

**Taints** — Applied to nodes to repel pods unless the pod explicitly tolerates them.

**Tolerations** — Applied to pods to allow scheduling on tainted nodes.

```bash
kubectl taint nodes node1 gpu=true:NoSchedule
```

```yaml
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

**Effects:** `NoSchedule`, `PreferNoSchedule`, `NoExecute`

---

### 32. What is Node Affinity and Pod Anti-Affinity?
**Answer:**

**Node Affinity** — Schedule pods on specific nodes based on node labels.

**Pod Anti-Affinity** — Spread pods across nodes/zones for high availability.

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: my-app
      topologyKey: kubernetes.io/hostname   # Spread across nodes
```

---

### 33. What are Liveness, Readiness, and Startup Probes?
**Answer:**

| Probe | Purpose | On Failure |
|---|---|---|
| **Liveness** | Is the container still running correctly? | Restart container |
| **Readiness** | Is the container ready to serve traffic? | Remove from Service endpoints |
| **Startup** | Has the app started? (for slow starters) | Kill and restart after timeout |

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

### 34. What is a PodDisruptionBudget (PDB)?
**Answer:**
A PDB limits the number of pods that can be voluntarily disrupted at a time (during node drains, upgrades) to ensure availability.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2     # Always keep at least 2 pods running
  selector:
    matchLabels:
      app: my-app
```

---

## 🟣 SECTION 6: ADVANCED CONCEPTS

---

### 35. What is Helm?
**Answer:**
Helm is the package manager for Kubernetes. Charts are packages of pre-configured Kubernetes resources.

```bash
helm install my-release bitnami/nginx        # Install
helm upgrade my-release bitnami/nginx        # Upgrade
helm rollback my-release 1                   # Rollback
helm list                                    # List releases
helm uninstall my-release                    # Delete
```

---

### 36. What is a Custom Resource Definition (CRD)?
**Answer:**
CRDs extend the Kubernetes API with custom resource types. Once defined, you manage them like any native Kubernetes object. Used by Operators (Prometheus, Cert-Manager, ArgoCD) to add domain-specific functionality.

---

### 37. What is a Kubernetes Operator?
**Answer:**
An Operator packages, deploys, and manages a Kubernetes application using custom controllers + CRDs. Operators encode operational knowledge (how to install, upgrade, backup, recover a stateful app).

Examples: Prometheus Operator, Cert-Manager, Strimzi (Kafka), CloudNativePG (PostgreSQL).

---

### 38. What is the difference between `kubectl apply` and `kubectl create`?
**Answer:**

| Command | Behavior | Idempotent |
|---|---|---|
| `kubectl create` | Creates resource. Fails if already exists. | No |
| `kubectl apply` | Creates or updates. Tracks changes. | Yes |
| `kubectl replace` | Replaces entirely. Fails if not exists. | No |

Always prefer `kubectl apply` in CI/CD workflows.

---

### 39. What are Deployment strategies?
**Answer:**

| Strategy | How | Downtime | Notes |
|---|---|---|---|
| **Recreate** | Kill all old, start new | Yes | Simplest |
| **RollingUpdate** | Replace pods gradually | No | Default |
| **Blue-Green** | Switch traffic between two environments | No | 2x resource cost |
| **Canary** | Send % of traffic to new version | No | Lowest risk |

Blue-Green and Canary use Ingress weight rules, Argo Rollouts, or Istio.

---

### 40. What is a Service Mesh (Istio)?
**Answer:**
Istio is a service mesh that injects an Envoy sidecar proxy into every pod. It provides:
- Traffic management (canary, circuit breaking, retries)
- Security (mTLS between services)
- Observability (tracing, metrics, logs)
- Policy enforcement (rate limiting, access control)

---

### 41. What is GitOps? What are ArgoCD and Flux?
**Answer:**
GitOps makes Git the single source of truth for cluster state. ArgoCD and Flux:
- Watch a Git repo for changes
- Automatically sync the cluster to match Git
- Detect and correct drift
- Enable rollback by reverting git commits

---

### 42. What is the Kubernetes control/reconciliation loop?
**Answer:**
Controllers continuously run a reconciliation loop:
1. **Observe** — Read current cluster state
2. **Diff** — Compare to desired state
3. **Act** — Take corrective action

This is why Kubernetes is self-healing — it constantly works to match desired state.

---

### 43. What is `kube-proxy`?
**Answer:**
`kube-proxy` runs on every node and maintains iptables/IPVS rules that route traffic from Service ClusterIPs to actual pod IPs. It watches the API server for Service and Endpoint changes.

**Modes:** iptables (default), IPVS (better performance at scale), userspace (legacy)

---

### 44. What is Cluster Autoscaler?
**Answer:**
Cluster Autoscaler automatically adjusts the number of nodes in a cluster:
- Scales **up** when pods are pending due to insufficient resources
- Scales **down** when nodes are underutilized for a period

Works with cloud providers (AWS EKS, GKE, AKS) to add/remove VMs.

---

### 45. What is the difference between ConfigMap volume mount and environment variable injection?
**Answer:**

| Method | Auto-updates? | Use Case |
|---|---|---|
| **Volume mount** | Yes (~1 min) | Config files the app watches |
| **Env variable** | No (needs restart) | Static config at startup |

For secrets: env vars are slightly less secure (visible in `kubectl describe`). Volume mounts are preferred for sensitive data.

---

## ⚫ SECTION 7: SCENARIO-BASED QUESTIONS

---

### 46. SCENARIO: Pod stuck in `CrashLoopBackOff`. How do you debug?

**Answer:**
```bash
# Step 1: Check events
kubectl describe pod <pod-name> -n <namespace>

# Step 2: Check current logs
kubectl logs <pod-name> -n <namespace>

# Step 3: Check previous container logs
kubectl logs <pod-name> --previous -n <namespace>

# Step 4: Decode the exit code
# Exit 1   = App error / missing config
# Exit 137 = OOMKilled (increase memory limits)
# Exit 139 = Segfault
# Exit 143 = SIGTERM

# Step 5: Override command to prevent crash and shell in
kubectl run debug --image=my-app:latest --command -- sleep 3600
kubectl exec -it debug -- bash
```

**Common causes:** Bad config, missing environment variable, OOMKilled, failed liveness probe, missing dependency.

---

### 47. SCENARIO: Pod stuck in `Pending`. What do you check?

**Answer:**
```bash
kubectl describe pod <pod-name>
# Focus on the Events section at the bottom
```

| Event Message | Cause | Fix |
|---|---|---|
| `Insufficient cpu/memory` | No node has capacity | Add nodes or reduce requests |
| `node(s) had taints that pod didn't tolerate` | Taint mismatch | Add tolerations to pod |
| `didn't match node affinity` | Label mismatch | Fix affinity rules |
| `PVC not bound` | PVC waiting for PV | Check StorageClass, create PV |
| `ImagePullBackOff` | Can't pull image | Fix image name/tag/credentials |

```bash
kubectl get nodes                    # Check node status
kubectl get pvc -n <namespace>       # Check PVC binding
```

---

### 48. SCENARIO: New deployment broke production. How do you rollback?

**Answer:**
```bash
# Check rollout history
kubectl rollout history deployment/my-app

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to specific revision
kubectl rollout undo deployment/my-app --to-revision=3

# Monitor rollback
kubectl rollout status deployment/my-app

# Verify image version
kubectl describe deployment my-app | grep Image
```

---

### 49. SCENARIO: A service is not reachable inside the cluster. How do you troubleshoot?

**Answer:**
```bash
# Step 1: Check pods are running
kubectl get pods -l app=my-app

# Step 2: Check service and endpoints (empty endpoints = selector mismatch)
kubectl get endpoints my-service

# Step 3: Compare service selector vs pod labels
kubectl describe svc my-service     # Check selector
kubectl get pods --show-labels       # Check labels

# Step 4: Test from another pod
kubectl run test --image=busybox -it --rm -- wget -qO- http://my-service:80

# Step 5: Check NetworkPolicy blocking traffic
kubectl get networkpolicy -n <namespace>
```

---

### 50. SCENARIO: Node is NotReady. What do you do?

**Answer:**
```bash
# Check node status
kubectl describe node <node-name>
# Look at Conditions: MemoryPressure, DiskPressure, PIDPressure

# SSH into node and check kubelet
systemctl status kubelet
journalctl -u kubelet -n 100

# If node is truly broken, drain it
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# Cordon to prevent new scheduling
kubectl cordon <node>

# After fix, restore
kubectl uncordon <node>
```

---

### 51. SCENARIO: Application is slow — high latency despite pods running. How to investigate?

**Answer:**
```bash
# Check pod resource usage
kubectl top pods -n <namespace>
kubectl top nodes

# Check HPA — is it scaling or throttling?
kubectl describe hpa my-app-hpa

# Check if OOMKilled (memory issues)
kubectl describe pods | grep -A3 "Last State"

# Check CPU limits causing throttling
kubectl describe pod <pod> | grep -A6 "Limits"

# Check endpoints — all traffic hitting too few pods?
kubectl get endpoints my-service
```

---

### 52. SCENARIO: Debug a running pod without modifying it.

**Answer:**
```bash
# Kubernetes 1.23+: Ephemeral debug container
kubectl debug -it <pod-name> --image=busybox --target=my-container

# Create a copy of the pod with debug tools
kubectl debug <pod-name> -it --copy-to=debug-pod --image=ubuntu

# Debug images
# busybox    — Basic tools
# nicolaka/netshoot — Full network toolkit (curl, dig, tcpdump)
# alpine     — Lightweight with apk
```

---

### 53. SCENARIO: Migrate workloads to a new node pool with zero downtime.

**Answer:**
```bash
# Step 1: Taint old nodes — prevent new pods scheduling there
kubectl taint nodes <old-node> maintenance=true:NoSchedule

# Step 2: Drain old nodes one by one (PDB protects availability)
kubectl drain <old-node-1> --ignore-daemonsets --delete-emptydir-data

# Step 3: Verify pods are on new nodes
kubectl get pods -o wide

# Step 4: Repeat for all old nodes

# Step 5: Remove old nodes
kubectl delete node <old-node-1>
```

---

### 54. SCENARIO: Rotate a secret without restarting pods.

**Answer:**
```bash
# Update secret
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=newpassword \
  --dry-run=client -o yaml | kubectl apply -f -
```

- **Volume-mounted secrets** auto-update in ~1 minute if app watches the file
- **Environment variable secrets** require a pod restart:
  ```bash
  kubectl rollout restart deployment/my-app
  ```

Best practice: Use External Secrets Operator with AWS Secrets Manager or HashiCorp Vault for automatic rotation.

---

### 55. SCENARIO: How do you set up zero-downtime deployments?

**Answer:**

**1. Rolling update with maxUnavailable=0:**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

**2. Readiness probe — traffic only goes to ready pods:**
```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
```

**3. Graceful shutdown — drain in-flight requests:**
```yaml
lifecycle:
  preStop:
    exec:
      command: ["/bin/sh", "-c", "sleep 5"]
terminationGracePeriodSeconds: 30
```

**4. PodDisruptionBudget — always keep minimum pods:**
```yaml
spec:
  minAvailable: 2
```

---

### 56. SCENARIO: Cluster certificate is expiring soon. How do you renew?

**Answer:**
```bash
# Check expiry
kubeadm certs check-expiration

# Renew all certificates
kubeadm certs renew all

# Restart control plane components
kubectl -n kube-system delete pod -l component=kube-apiserver
kubectl -n kube-system delete pod -l component=kube-controller-manager
kubectl -n kube-system delete pod -l component=kube-scheduler
```

Prevention: Monitor cert expiry with Prometheus alerts. Managed clusters (EKS, GKE) auto-renew.

---

### 57. SCENARIO: How do you limit a namespace to 4 CPUs and 8GB RAM?

**Answer:**
```yaml
# ResourceQuota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: team-alpha
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"

---
# LimitRange — defaults so pods without limits still count
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-alpha
spec:
  limits:
  - default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 250m
      memory: 128Mi
    type: Container
```

---

### 58. SCENARIO: How do you expose a service to the internet with TLS?

**Answer:**

**Step 1: Install cert-manager**
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

**Step 2: Create ClusterIssuer (Let's Encrypt)**
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@mycompany.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

**Step 3: Ingress with TLS**
```yaml
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.mycompany.com
    secretName: api-tls-secret   # cert-manager auto-fills this
```

---

### 59. SCENARIO: Multiple teams share one cluster. How do you isolate them?

**Answer:**
```bash
# 1. Separate namespaces
kubectl create namespace team-alpha
kubectl create namespace team-beta

# 2. ResourceQuota per namespace
# 3. NetworkPolicy — deny cross-namespace traffic
# 4. RBAC — teams only access their namespace
# 5. LimitRange — enforce resource defaults
# 6. Pod Security Standards — enforce restricted/baseline
```

```yaml
# NetworkPolicy: only same-namespace pods can talk to each other
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}    # Only from pods in same namespace
```

---

### 60. SCENARIO: How do you implement a canary deployment?

**Answer:**

**Method 1 — Replica-based (simple):**
- v1 Deployment: 9 replicas (90% traffic)
- v2 Deployment: 1 replica (10% traffic)
- Both match the same Service selector

**Method 2 — Nginx Ingress weight:**
```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"   # 10% to canary
```

**Method 3 — Argo Rollouts (production-grade):**
```yaml
strategy:
  canary:
    steps:
    - setWeight: 10
    - pause: {duration: 10m}
    - setWeight: 50
    - pause: {duration: 10m}
    - setWeight: 100
```

---

### 61. SCENARIO: How do you troubleshoot ImagePullBackOff?

**Answer:**
```bash
kubectl describe pod <pod-name>
# Look for: Back-off pulling image, ErrImagePull

# Common causes and fixes:

# 1. Wrong image name or tag
kubectl set image deployment/my-app my-app=myrepo/myimage:correct-tag

# 2. Private registry — missing imagePullSecret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass

# 3. Reference the secret in the pod
spec:
  imagePullSecrets:
  - name: regcred

# 4. Image doesn't exist — verify
docker pull myrepo/myimage:tag
```

---

### 62. SCENARIO: How do you force delete a pod stuck in Terminating state?

**Answer:**
```bash
# Normal delete with grace period (try this first)
kubectl delete pod <pod-name> --grace-period=0

# Force delete (bypasses graceful shutdown)
kubectl delete pod <pod-name> --grace-period=0 --force

# If still stuck — check if node is unreachable
kubectl get nodes
# If node is NotReady, the pod will be stuck until node recovers or is deleted
kubectl delete node <node-name>   # Removes node from cluster
```

**Warning:** Force deletion doesn't guarantee the container actually stopped — use carefully for stateful apps.

---

### 63. SCENARIO: How do you perform a blue-green deployment in Kubernetes?

**Answer:**
```yaml
# Blue (current) Deployment — 3 replicas with label version: blue
# Green (new) Deployment — 3 replicas with label version: green

# Service points to blue initially
spec:
  selector:
    app: my-app
    version: blue    # <-- traffic goes here

# After testing green, switch Service selector
kubectl patch service my-service -p '{"spec":{"selector":{"version":"green"}}}'

# If green is good, delete blue
kubectl delete deployment my-app-blue

# If green is bad, switch back instantly
kubectl patch service my-service -p '{"spec":{"selector":{"version":"blue"}}}'
```

---

### 64. SCENARIO: etcd is running out of disk space. What do you do?

**Answer:**
```bash
# Check etcd size
ETCDCTL_API=3 etcdctl endpoint status --write-out=table

# Compact the revision history
ETCDCTL_API=3 etcdctl compact $(etcdctl endpoint status --write-out=json | jq '.[0].Status.header.revision')

# Defragment to reclaim disk space
ETCDCTL_API=3 etcdctl defrag --endpoints=https://127.0.0.1:2379

# Increase etcd quota (default 2GB)
# In etcd config: --quota-backend-bytes=8589934592  (8GB)

# Best prevention: take regular snapshots and clean old ones
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d).db
```

---

### 65. SCENARIO: How do you schedule a pod only on nodes with GPUs?

**Answer:**

**Method 1 — Node Selector (simple):**
```yaml
spec:
  nodeSelector:
    accelerator: nvidia-gpu
```

**Method 2 — Node Affinity (flexible):**
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-gpu
```

**Method 3 — Taints + Tolerations (preferred for GPU isolation):**
```bash
# Taint GPU nodes
kubectl taint nodes gpu-node-1 nvidia.com/gpu=true:NoSchedule
```
```yaml
# Only GPU pods can schedule there
spec:
  tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
  resources:
    limits:
      nvidia.com/gpu: 1    # Request 1 GPU
```

---

*© Kubernetes Interview Q&A — 65 Questions*
*Sections: Basics · Architecture · Workloads · Networking · Storage · Security · Advanced · 20 Scenarios*
