# Appendix: Quick Reference

## Key Terms at a Glance

### Physical Infrastructure

| Term | One-Line Definition |
|------|---------------------|
| **Data Center** | A facility housing servers with dedicated power, cooling, security, and network connectivity |
| **Rack (42U)** | Standardized cabinet holding servers; "U" is 1.75 inches of vertical space |
| **UPS** | Uninterruptible Power Supply—batteries that keep servers running during power outages |
| **BMC/IPMI** | Out-of-band management allowing remote control of servers without an OS |
| **ECC RAM** | Error-Correcting Code memory that detects and fixes single-bit errors |
| **Hot-swap** | Ability to replace components (like disks) without shutting down the system |

### Virtualization

| Term | One-Line Definition |
|------|---------------------|
| **Hypervisor** | Software that creates and manages virtual machines |
| **Type 1 Hypervisor** | Runs directly on hardware (ESXi, Hyper-V, KVM)—used in production |
| **Type 2 Hypervisor** | Runs on a host OS (VirtualBox, VMware Workstation)—used for development |
| **Virtual Machine** | A software emulation of a complete computer with its own OS |
| **Guest OS** | The operating system running inside a virtual machine |
| **Host** | The physical machine running the hypervisor |
| **Snapshot** | A point-in-time copy of a VM's state that can be restored |
| **Live Migration** | Moving a running VM between physical hosts with zero downtime |

### Operating System

| Term | One-Line Definition |
|------|---------------------|
| **Kernel** | The core of the OS with direct hardware access and resource management |
| **User Space** | Where applications run, isolated from direct hardware access |
| **System Call** | The interface for user space programs to request kernel services |
| **Process** | A running program with its own isolated memory space and resources |
| **Thread** | A unit of execution within a process; threads share memory |
| **PID** | Process ID—unique identifier for a running process |
| **Virtual Memory** | Abstraction giving each process the illusion of contiguous memory |

### Networking

| Term | One-Line Definition |
|------|---------------------|
| **IP Address** | Numerical identifier for a device on a network (e.g., 192.168.1.100) |
| **Port** | Number (0-65535) identifying a specific application on a host |
| **Socket** | Combination of IP address and port representing a network endpoint |
| **DNS** | Domain Name System—translates hostnames to IP addresses |
| **TTL** | Time To Live—how long DNS records should be cached |
| **Load Balancer** | Distributes traffic across multiple backend servers |
| **Layer 4 LB** | Load balances based on IP/port (fast, no content inspection) |
| **Layer 7 LB** | Load balances based on HTTP content (flexible, slight overhead) |
| **NAT** | Network Address Translation—maps private IPs to public IPs |

### Containers

| Term | One-Line Definition |
|------|---------------------|
| **Container** | Isolated environment sharing the host kernel; lighter than VMs |
| **Image** | Read-only template containing application and dependencies |
| **Dockerfile** | Text file with instructions for building a container image |
| **Registry** | Storage for container images (Docker Hub, ECR, GCR) |
| **Volume** | Persistent storage that survives container restarts |
| **Namespace** | Linux feature providing isolation (PID, network, mount, etc.) |
| **cgroup** | Control group—limits resource usage (CPU, memory, I/O) |

### Kubernetes

| Term | One-Line Definition |
|------|---------------------|
| **Cluster** | A set of nodes running containerized applications managed by Kubernetes |
| **Control Plane** | Components that manage the cluster (API server, scheduler, etcd) |
| **Node** | A worker machine (VM or physical) in a Kubernetes cluster |
| **Pod** | Smallest deployable unit; one or more containers sharing network/storage |
| **Deployment** | Declares desired state for pods; handles scaling and updates |
| **Service** | Stable network endpoint that load balances across pods |
| **ConfigMap** | Non-sensitive configuration data stored in the cluster |
| **Secret** | Sensitive data (passwords, keys) stored in the cluster |
| **Ingress** | Routes external HTTP/HTTPS traffic to services |
| **kubectl** | Command-line tool for interacting with Kubernetes |

---

## Essential Commands

### Docker

```bash
# Images
docker build -t myapp:1.0 .          # Build image from Dockerfile
docker images                         # List images
docker pull nginx:latest              # Download image
docker push myregistry/myapp:1.0      # Upload image

# Containers
docker run -d -p 8080:80 nginx        # Run container (detached, port mapped)
docker ps                             # List running containers
docker ps -a                          # List all containers
docker logs <container>               # View container logs
docker exec -it <container> sh        # Shell into running container
docker stop <container>               # Stop container
docker rm <container>                 # Remove container

# Cleanup
docker system prune                   # Remove unused data
```

### Kubernetes (kubectl)

```bash
# Cluster info
kubectl cluster-info                  # Display cluster endpoint
kubectl get nodes                     # List nodes

# Workloads
kubectl get pods                      # List pods
kubectl get deployments               # List deployments
kubectl get services                  # List services
kubectl describe pod <name>           # Detailed pod info
kubectl logs <pod>                    # View pod logs
kubectl logs -f <pod>                 # Follow logs

# Apply/Delete
kubectl apply -f manifest.yaml        # Create/update resources
kubectl delete -f manifest.yaml       # Delete resources

# Debugging
kubectl exec -it <pod> -- sh          # Shell into pod
kubectl port-forward <pod> 8080:80    # Forward local port to pod

# Scaling
kubectl scale deployment <name> --replicas=5
```

### Linux Networking

```bash
# Check listening ports
ss -tlnp                              # TCP ports with process names
netstat -tlnp                         # Alternative to ss

# DNS
dig example.com                       # DNS lookup
nslookup example.com                  # Alternative DNS lookup

# Connectivity
ping 8.8.8.8                          # ICMP connectivity test
curl -v http://example.com            # HTTP request with details
telnet host 80                        # Test TCP port connectivity
```

---

## Suggested Resources for Deeper Learning

### Books

- **"The Phoenix Project"** by Gene Kim - Novel about DevOps transformation
- **"Site Reliability Engineering"** by Google - SRE practices at scale
- **"Kubernetes Up & Running"** by Kelsey Hightower - Practical K8s guide
- **"Docker Deep Dive"** by Nigel Poulton - Comprehensive Docker coverage

### Online Resources

- **Kubernetes Documentation** (kubernetes.io/docs) - Official reference
- **Docker Documentation** (docs.docker.com) - Official Docker guides
- **The Twelve-Factor App** (12factor.net) - Methodology for cloud-native apps
- **Linux Journey** (linuxjourney.com) - Learn Linux fundamentals

### Practice Environments

- **Katacoda** - Browser-based interactive tutorials
- **Play with Docker** (play-with-docker.com) - Free Docker environment
- **Minikube** - Local Kubernetes cluster for learning
- **kind** (Kubernetes in Docker) - Another local K8s option

### Certifications (If Your Career Path Requires Them)

- **CKA** - Certified Kubernetes Administrator
- **CKAD** - Certified Kubernetes Application Developer
- **AWS/GCP/Azure** - Cloud provider certifications

---

## The City Analogy Recap

| Infrastructure Layer | City Equivalent | Chapter |
|---------------------|-----------------|---------|
| Data Center | Industrial District | 1 |
| Server Rack | Building | 1 |
| Physical Server | Floor of Building | 1 |
| Virtual Machine | Apartment | 2 |
| Container | Room | 5 |
| Process | Person | 3 |
| Network Port | Apartment Door Number | 4 |
| Load Balancer | Traffic Cop / Receptionist | 4 |
| Kubernetes | Building Management Company | 6 |

---

*Thank you for reading "From Bare Metal to Containers." Understanding these fundamentals will serve you well as you encounter new technologies and make infrastructure decisions in your career.*
