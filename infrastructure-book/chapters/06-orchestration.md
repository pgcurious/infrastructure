# Chapter 6: Orchestration - Managing at Scale

*Problem: You've containerized your applications. But now you have 200 containers across 50 servers. How do you deploy them? What happens when a server fails? How do containers find each other? How do you update without downtime? Manual management doesn't scale.*

---

## The Container Sprawl Problem

Docker made it easy to run one container. But real applications aren't one container:

- Web frontend (3 instances for redundancy)
- API gateway (2 instances)
- User service (3 instances)
- Order service (3 instances)
- Inventory service (2 instances)
- Database (primary + replica)
- Cache (Redis cluster, 3 nodes)
- Message queue (3 nodes)

That's already 20+ containers for a modest microservices application. At scale, companies run thousands or tens of thousands.

Without automation, you face:
- **Deployment**: Which container goes on which server?
- **Scaling**: Traffic spike? Manually start more containers?
- **Failure recovery**: Server dies? Manually move containers?
- **Service discovery**: How do services find each other's changing IP addresses?
- **Load balancing**: How is traffic distributed?
- **Updates**: How do you roll out new versions without downtime?

This is where **container orchestration** comes in.

## Kubernetes: The Industry Standard

**Kubernetes** (K8s) is an open-source container orchestration platform. Originally developed by Google based on their internal system (Borg), it's now the de facto standard for running containers at scale.

Kubernetes handles:
- **Scheduling**: Deciding which node runs which container
- **Self-healing**: Restarting failed containers, replacing unresponsive nodes
- **Service discovery**: DNS-based discovery between services
- **Load balancing**: Distributing traffic across container instances
- **Rolling updates**: Zero-downtime deployments
- **Configuration management**: Separating config from code
- **Secret management**: Secure handling of passwords, API keys

### Kubernetes Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         Control Plane                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ API Server  │  │ Scheduler   │  │ Controller  │  ┌───────┐  │
│  │             │  │             │  │ Manager     │  │ etcd  │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └───────┘  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
     ┌──────────┐    ┌──────────┐    ┌──────────┐
     │  Node 1  │    │  Node 2  │    │  Node 3  │
     │ ┌──────┐ │    │ ┌──────┐ │    │ ┌──────┐ │
     │ │kubelet│ │    │ │kubelet│ │    │ │kubelet│ │
     │ └──────┘ │    │ └──────┘ │    │ └──────┘ │
     │ ┌──────┐ │    │ ┌──────┐ │    │ ┌──────┐ │
     │ │Pods  │ │    │ │Pods  │ │    │ │Pods  │ │
     │ └──────┘ │    │ └──────┘ │    │ └──────┘ │
     └──────────┘    └──────────┘    └──────────┘
         Worker Nodes (where containers actually run)
```

**Control Plane** (the brain):
- **API Server**: The front door; all communication goes through here
- **Scheduler**: Decides which node should run new pods
- **Controller Manager**: Ensures desired state matches actual state
- **etcd**: Distributed key-value store holding all cluster data

**Worker Nodes** (the muscle):
- **kubelet**: Agent that manages containers on the node
- **Container Runtime**: Actually runs containers (containerd, CRI-O)
- **kube-proxy**: Handles network routing

## Core Kubernetes Concepts

You only need to understand a handful of concepts to work with Kubernetes.

### Pod: The Smallest Unit

A **Pod** is one or more containers that:
- Share the same network namespace (localhost works between them)
- Share storage volumes
- Are scheduled together on the same node
- Have the same lifecycle

Most pods contain a single container. Multi-container pods are for tightly coupled processes (like a main app with a log shipper sidecar).

```yaml
# Example: A simple pod (you usually don't create pods directly)
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: mycompany/myapp:1.0
    ports:
    - containerPort: 8080
```

### Deployment: Declarative Pod Management

A **Deployment** declares your desired state: "I want 3 replicas of this pod running." Kubernetes makes it happen and keeps it that way.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-service
spec:
  replicas: 3                    # Keep 3 instances running
  selector:
    matchLabels:
      app: orders-service        # Manage pods with this label
  template:
    metadata:
      labels:
        app: orders-service
    spec:
      containers:
      - name: orders
        image: mycompany/orders-service:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:              # Minimum resources needed
            memory: "256Mi"
            cpu: "250m"
          limits:                # Maximum resources allowed
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:          # Is the container healthy?
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:         # Is the container ready for traffic?
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

When you apply this, Kubernetes:
1. Creates 3 pods matching the template
2. Monitors them continuously
3. If a pod crashes, starts a replacement
4. If a node fails, reschedules pods elsewhere
5. If you change the image tag, performs a rolling update

### Service: Stable Network Endpoint

Pods come and go. Their IP addresses change. A **Service** provides a stable endpoint.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: orders-service
spec:
  selector:
    app: orders-service          # Route to pods with this label
  ports:
  - port: 80                     # Service port
    targetPort: 8080             # Pod port
  type: ClusterIP                # Internal to cluster (default)
```

```
                  ┌─────────────────────────────────┐
                  │      Service: orders-service    │
                  │      IP: 10.96.45.123:80        │
                  │      DNS: orders-service.default│
                  └───────────────┬─────────────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                    ▼             ▼             ▼
              ┌──────────┐ ┌──────────┐ ┌──────────┐
              │  Pod 1   │ │  Pod 2   │ │  Pod 3   │
              │10.244.1.5│ │10.244.2.8│ │10.244.3.2│
              └──────────┘ └──────────┘ └──────────┘
```

Other services can now reach orders-service via:
- DNS: `orders-service.default.svc.cluster.local` (or just `orders-service`)
- ClusterIP: `10.96.45.123`

The Service handles load balancing automatically.

### Service Types

| Type | Use Case |
|------|----------|
| **ClusterIP** | Internal traffic only (default) |
| **NodePort** | Expose on each node's IP at a static port (30000-32767) |
| **LoadBalancer** | Provision external load balancer (cloud environments) |

```yaml
# Expose to internet via cloud load balancer
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
spec:
  type: LoadBalancer
  selector:
    app: api-gateway
  ports:
  - port: 443
    targetPort: 8443
```

## Configuration and Secrets

### ConfigMap: Non-Sensitive Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  SPRING_PROFILES_ACTIVE: "production"
  LOG_LEVEL: "INFO"
  FEATURE_FLAG_NEW_UI: "true"
```

Use in a Deployment:
```yaml
spec:
  containers:
  - name: myapp
    envFrom:
    - configMapRef:
        name: app-config
```

### Secret: Sensitive Data

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: cG9zdGdyZXM=           # base64 encoded
  password: c3VwZXJzZWNyZXQ=       # base64 encoded
```

```bash
# Create secret from command line
kubectl create secret generic db-credentials \
  --from-literal=username=postgres \
  --from-literal=password=supersecret
```

> **Think About It:** Secrets in Kubernetes are base64 encoded, not encrypted by default. What additional measures might you take for truly sensitive data?

## Rolling Updates and Rollbacks

When you update a Deployment's image, Kubernetes performs a rolling update:

```bash
# Update the image
kubectl set image deployment/orders-service \
  orders=mycompany/orders-service:2.0

# Or edit the YAML and apply
kubectl apply -f deployment.yaml
```

```
Rolling Update Sequence:

1. Start: 3 pods running v1.0
   [v1] [v1] [v1]

2. Create new pod with v2.0
   [v1] [v1] [v1] [v2-starting]

3. v2.0 pod passes readiness probe
   [v1] [v1] [v1] [v2-ready]

4. Terminate one v1.0 pod
   [v1] [v1] [v2] [v1-terminating]

5. Repeat until all pods are v2.0
   [v2] [v2] [v2]
```

If something goes wrong:
```bash
# Check rollout status
kubectl rollout status deployment/orders-service

# Undo the rollout
kubectl rollout undo deployment/orders-service

# Roll back to specific revision
kubectl rollout undo deployment/orders-service --to-revision=2
```

## A Complete Example

Let's put it together for a Spring Boot service:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-service
  labels:
    app: orders-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: orders-service
  template:
    metadata:
      labels:
        app: orders-service
    spec:
      containers:
      - name: orders
        image: mycompany/orders-service:1.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "kubernetes"
        - name: DATABASE_HOST
          value: "postgres-service"
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: orders-service
spec:
  selector:
    app: orders-service
  ports:
  - port: 80
    targetPort: 8080
```

```bash
# Deploy
kubectl apply -f deployment.yaml

# Check status
kubectl get deployments
kubectl get pods
kubectl get services

# View logs
kubectl logs -f deployment/orders-service

# Scale up
kubectl scale deployment orders-service --replicas=5
```

## When You Don't Need Kubernetes

Kubernetes is powerful but complex. You might not need it if:

- You have a single application or small number of services
- Your team doesn't have Kubernetes expertise
- You're running on a single server
- Your deployment frequency is low
- A simpler solution (Docker Compose, ECS, managed PaaS) meets your needs

Signs you might need Kubernetes:
- Many microservices with complex interdependencies
- Need for automatic scaling based on load
- Require zero-downtime deployments
- Running across multiple nodes for high availability
- Team has or can acquire Kubernetes expertise

Many organizations use managed Kubernetes (EKS, GKE, AKS) to reduce operational burden.

---

## Key Takeaways

1. **Kubernetes automates what you'd otherwise do manually** - Scheduling, scaling, self-healing, service discovery, and rolling updates are handled declaratively. You describe what you want, Kubernetes makes it happen.

2. **Three concepts cover most use cases: Pod, Deployment, Service** - Pods are the atomic unit. Deployments manage pod lifecycles and updates. Services provide stable network endpoints. Master these first.

3. **Kubernetes is powerful but not always necessary** - The complexity is justified at scale or when you need its specific features. For simpler scenarios, Docker Compose or a managed container service may be better choices.

---

*Next Chapter: [Putting It All Together](07-putting-it-together.md) - Tracing a request through all the layers*
