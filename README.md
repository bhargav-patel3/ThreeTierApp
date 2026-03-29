# Three-Tier To-Do Application

A containerized three-tier application deployed on AWS EKS with comprehensive DevOps practices.

## Project Overview

This is a full-stack MERN (MongoDB, Express, React, Node.js) application designed to demonstrate modern DevOps practices including containerization, Kubernetes orchestration, and cloud infrastructure on AWS.

## Architecture

### Application Layers

1. **Frontend** - React.js SPA served via Nginx
2. **Backend** - Node.js/Express REST API
3. **Database** - MongoDB with persistent storage

### Infrastructure

- **Container Orchestration**: AWS EKS (Elastic Kubernetes Service)
- **Ingress Controller**: NGINX Ingress with AWS Network Load Balancer
- **Storage**: Persistent Volumes (PV) and Persistent Volume Claims (PVC)
- **Container Registry**: Docker Hub

## Tech Stack

### Application
- **Frontend**: React.js, Material-UI
- **Backend**: Node.js, Express.js
- **Database**: MongoDB 4.4.6

### DevOps & Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes (EKS)
- **Network**: NGINX Ingress Controller
- **Cloud Provider**: AWS


## Deployment Details

### Folder Structure

```
Application-Code/
├── frontend/          # React application
│   ├── src/
│   ├── public/
│   └── Dockerfile     # Multi-stage, optimized for production
└── backend/           # Node.js Express API
    ├── routes/
    ├── models/
    ├── db.js
    └── Dockerfile

k8s/
├── backend-deployment.yml
├── backend-service.yml
├── frontend-deployment.yml
├── frontend-service.yml
├── mongo-deployment.yml
├── mongo-service.yml
├── ingress.yml
├── pv.yml
├── pvc.yml
├── pvc.yml
├── namespace.yml
└── secrets.yml
```

### Kubernetes Deployment

#### Key Configurations

- **Database Authentication**: Kubernetes Secrets for MongoDB credentials
- **Persistent Storage**: MongoDB data persisted via PVs
- **Load Balancing**: AWS Network Load Balancer (ELB) via Ingress

### Environment variables

Backend deployment uses:
- `MONGO_CONN_STR`: Connection string to MongoDB
- `USE_DB_AUTH`: Enables MongoDB authentication
- `MONGO_USERNAME` & `MONGO_PASSWORD`: Injected from Kubernetes Secrets


## How It Works

### Deployment Flow

1. **Docker Images** built and pushed to Docker Hub
2. **Kubernetes manifests** applied to EKS cluster
3. **NGINX Ingress Controller** routes external traffic to services
4. **AWS ELB** provides public endpoint with DNS name
5. **Frontend** makes requests to `/api/tasks` which routes through Ingress to Backend
6. **Backend** queries MongoDB with authenticated credentials from Secrets

### Access

Once deployed, the application is accessible via:
```
http://<AWS-ELB-DNS-NAME>
```

Run to get the DNS name:
```bash
kubectl get ingress -n three-tier
```

## Deployment Instructions

### Prerequisites

- AWS EKS cluster running
- `kubectl` configured to access EKS cluster
- NGINX Ingress Controller installed:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace --set controller.service.type=LoadBalancer
```

### Deploy to Kubernetes

```bash
# Create namespace
kubectl apply -f k8s/namespace.yml

# Create MongoDB secrets
kubectl apply -f k8s/secrets.yml

# Deploy application
kubectl apply -f k8s/pv.yml
kubectl apply -f k8s/pvc.yml
kubectl apply -f k8s/mongo-deployment.yml
kubectl apply -f k8s/backend-deployment.yml
kubectl apply -f k8s/frontend-deployment.yml
kubectl apply -f k8s/ingress.yml

# Verify deployment
kubectl get all -n three-tier
kubectl get ingress -n three-tier
```

## Key DevOps Decisions

1. **Kubernetes Secrets**: Sensitive data (MongoDB credentials) managed securely
2. **Persistent Volumes**: Database data survives Pod restarts
3. **Rolling Updates**: RollingUpdate strategy for zero-downtime deployments
4. **Resource Limits**: CPU and memory constraints prevent resource exhaustion
5. **Multi-pod Backend**: 2 replicas for high availability
6. **Ingress Controller**: Single entry point for all traffic
7. **Containerization**: Application-agnostic deployment across environments

## Monitoring & Troubleshooting

### Check Pod Status
```bash
kubectl get pods -n three-tier
kubectl describe pod <pod-name> -n three-tier
```

### View Logs
```bash
kubectl logs -f <pod-name> -n three-tier
```

### Test Backend Connectivity
```bash
kubectl exec -it <frontend-pod> -n three-tier -- wget -qO- http://backend-svc:3500/api/tasks
```

### Get External IP
```bash
kubectl get svc -n ingress-nginx | grep ingress-nginx-controller
```

## Technologies Used

- **Cloud**: AWS (EKS, ELB)
- **Container Engine**: Docker
- **Orchestration**: Kubernetes
- **Ingress**: NGINX 
- **Language**: JavaScript (Node.js, React)
- **Database**: MongoDB

---

**Deployment Architecture**: Containerized → Docker Hub → EKS → NGINX Ingress → AWS ELB → Public Internet
