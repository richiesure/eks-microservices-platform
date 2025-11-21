# Microservices Platform on Amazon EKS

Production-grade Kubernetes deployment using Amazon EKS with Helm charts, auto-scaling, and load balancing.

## Architecture
```
                    ┌─────────────────────────────────────┐
                    │         Amazon EKS Cluster          │
                    │                                     │
┌──────────┐        │  ┌─────────┐      ┌─────────┐      │
│  Users   │───────▶│  │  Pod 1  │      │  Pod 2  │      │
└──────────┘        │  └────┬────┘      └────┬────┘      │
     │              │       │                │           │
     │              │       └───────┬────────┘           │
     │              │               │                    │
     │              │      ┌────────▼────────┐           │
     │              │      │  LoadBalancer   │           │
     └──────────────┼─────▶│    Service      │           │
                    │      └─────────────────┘           │
                    │                                     │
                    │  ┌──────────────────────────────┐  │
                    │  │  HorizontalPodAutoscaler     │  │
                    │  │  (2-10 pods, 70% CPU target) │  │
                    │  └──────────────────────────────┘  │
                    └─────────────────────────────────────┘
```

## Components

- **EKS Cluster**: Managed Kubernetes v1.28
- **Worker Nodes**: 2x t3.small instances
- **Container Registry**: Amazon ECR
- **Load Balancer**: AWS Classic Load Balancer
- **Auto-scaling**: HPA based on CPU utilization

## Quick Start
```bash
# Connect to cluster
aws eks update-kubeconfig --region eu-west-2 --name microservices-cluster

# Deploy with kubectl
kubectl apply -f k8s-manifests/

# Deploy with Helm
helm install web-app helm-chart/web-app --namespace devops-app
```

## Useful Commands
```bash
# View pods
kubectl get pods -n devops-app

# View logs
kubectl logs -f deployment/web-app -n devops-app

# Scale manually
kubectl scale deployment web-app -n devops-app --replicas=4

# Check auto-scaler
kubectl get hpa -n devops-app
```

## Endpoints

- **Application**: http://a9e6afc1d77fb48d494d7221e19038e3-847527286.eu-west-2.elb.amazonaws.com
- **Health Check**: http://[LB_URL]/health

## Cost

- EKS Control Plane: $0.10/hour (~$73/month)
- Worker Nodes (2x t3.small): $0.04/hour (~$60/month)
- Load Balancer: $0.025/hour (~$18/month)
- **Total**: ~$150/month (destroy after learning!)

## Cleanup
```bash
# Delete Helm release
helm uninstall web-app-helm -n helm-test

# Delete kubectl resources
kubectl delete namespace devops-app
kubectl delete namespace helm-test

# Delete EKS cluster
eksctl delete cluster --name microservices-cluster --region eu-west-2
```
