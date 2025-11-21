# Microservices Platform on Amazon EKS

Production-grade Kubernetes deployment migrating containerized applications from ECS to Amazon EKS with Helm charts, horizontal pod auto-scaling, and AWS load balancing.

## Project Overview

This project demonstrates the migration of a containerized web application from AWS ECS (Elastic Container Service) to Amazon EKS (Elastic Kubernetes Service), showcasing Kubernetes orchestration skills that are essential for modern DevOps roles.

## What Was Built

### Infrastructure Components
- **EKS Cluster**: Managed Kubernetes v1.28 control plane
- **Worker Nodes**: 2x t3.small EC2 instances in managed node group
- **Networking**: VPC with public/private subnets across 2 AZs
- **Load Balancer**: AWS Classic Load Balancer for external access
- **Container Registry**: Integration with existing ECR repository
- **Monitoring**: CloudWatch Container Insights enabled

### Kubernetes Resources
- **Deployment**: 2 replica pods with health checks
- **Service**: LoadBalancer type exposing port 80
- **HPA**: Horizontal Pod Autoscaler (2-10 pods, 70% CPU target)
- **Namespace**: Isolated `devops-app` namespace

### Helm Chart
- Templated deployment for repeatable installations
- Configurable values for different environments
- Production-ready with resource limits and probes

## Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        Amazon EKS                                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   Control Plane (AWS Managed)              │  │
│  │    API Server │ etcd │ Scheduler │ Controller Manager      │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌───────────────────────────┴───────────────────────────────┐  │
│  │                    Worker Nodes (t3.small x2)              │  │
│  │  ┌─────────────────┐              ┌─────────────────┐     │  │
│  │  │   Node 1        │              │   Node 2        │     │  │
│  │  │  ┌───────────┐  │              │  ┌───────────┐  │     │  │
│  │  │  │  Pod 1    │  │              │  │  Pod 2    │  │     │  │
│  │  │  │  web-app  │  │              │  │  web-app  │  │     │  │
│  │  │  └───────────┘  │              │  └───────────┘  │     │  │
│  │  └─────────────────┘              └─────────────────┘     │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌───────────────────────────┴───────────────────────────────┐  │
│  │              LoadBalancer Service (AWS ELB)                │  │
│  │         a9e6afc1d77fb...eu-west-2.elb.amazonaws.com       │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                         Internet Users
```

## Files Structure
```
eks-microservices-platform/
├── cluster-config.yaml      # eksctl cluster configuration
├── k8s-manifests/
│   ├── deployment.yaml      # Kubernetes deployment spec
│   ├── service.yaml         # LoadBalancer service
│   └── hpa.yaml             # Horizontal Pod Autoscaler
├── helm-chart/
│   └── web-app/
│       ├── Chart.yaml       # Helm chart metadata
│       ├── values.yaml      # Configurable values
│       └── templates/
│           ├── deployment.yaml
│           └── service.yaml
└── README.md
```

## Deployment Methods

### Method 1: Using kubectl (Direct)
```bash
# Connect to cluster
aws eks update-kubeconfig --region eu-west-2 --name microservices-cluster

# Create namespace
kubectl create namespace devops-app

# Apply manifests
kubectl apply -f k8s-manifests/
```

### Method 2: Using Helm (Recommended)
```bash
# Install with Helm
helm install web-app helm-chart/web-app --namespace devops-app --create-namespace

# Upgrade deployment
helm upgrade web-app helm-chart/web-app --namespace devops-app

# Rollback if needed
helm rollback web-app 1 --namespace devops-app
```

## Key Kubernetes Concepts Demonstrated

### Pod Management
- Multi-replica deployment for high availability
- Liveness probes to restart unhealthy containers
- Readiness probes to control traffic routing
- Resource requests and limits for QoS

### Service Discovery
- ClusterIP for internal communication
- LoadBalancer for external access
- DNS-based service discovery

### Auto-Scaling
- HPA based on CPU utilization
- Scale from 2 to 10 pods automatically
- Target 70% CPU utilization threshold

### Configuration Management
- Environment variables via Deployment spec
- Helm values for environment-specific configs
- Namespace isolation for multi-tenancy

## Useful Commands
```bash
# Cluster Operations
kubectl cluster-info
kubectl get nodes
eksctl get cluster --region eu-west-2

# Pod Operations
kubectl get pods -n devops-app
kubectl describe pod <pod-name> -n devops-app
kubectl logs -f <pod-name> -n devops-app
kubectl exec -it <pod-name> -n devops-app -- /bin/sh

# Scaling
kubectl scale deployment web-app -n devops-app --replicas=4
kubectl get hpa -n devops-app

# Debugging
kubectl get events -n devops-app --sort-by='.lastTimestamp'
kubectl top pods -n devops-app
kubectl top nodes
```

## Comparison: ECS vs EKS

| Feature | ECS (Day 3) | EKS (This Project) |
|---------|-------------|-------------------|
| Orchestrator | AWS Proprietary | Kubernetes (Industry Standard) |
| Learning Curve | Lower | Higher |
| Portability | AWS Only | Multi-cloud |
| Ecosystem | Limited | Vast (Helm, Operators, etc.) |
| Control | Less | Full Kubernetes API |
| Cost | Lower | Higher (~$73/mo control plane) |
| Job Market | Good | Excellent (80%+ job postings) |

## Cost Breakdown

| Resource | Hourly Cost | Monthly Cost |
|----------|-------------|--------------|
| EKS Control Plane | $0.10 | ~$73 |
| Worker Nodes (2x t3.small) | $0.042 | ~$61 |
| Load Balancer | $0.025 | ~$18 |
| Data Transfer | Variable | ~$5 |
| **Total** | **$0.167** | **~$157** |

**Note**: Delete cluster after learning to avoid ongoing costs!

## Cleanup
```bash
# Delete application resources
kubectl delete namespace devops-app

# Delete EKS cluster (takes 10-15 minutes)
eksctl delete cluster --name microservices-cluster --region eu-west-2

# Verify deletion
eksctl get cluster --region eu-west-2
```

## Skills Demonstrated

- Amazon EKS cluster provisioning with eksctl
- Kubernetes manifest creation (Deployment, Service, HPA)
- Helm chart development and templating
- Container orchestration and scaling
- Load balancer configuration
- kubectl command-line proficiency
- Infrastructure as Code for Kubernetes

## Next Steps

After mastering EKS basics, consider:
- Ingress controllers (NGINX, ALB Ingress)
- Service mesh (Istio, Linkerd)
- GitOps with ArgoCD or Flux
- Kubernetes RBAC and security
- Persistent storage with EBS CSI driver
