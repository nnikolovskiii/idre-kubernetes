# K8s-IDRE: Kubernetes LangGraph Application Stack

A complete Kubernetes deployment solution for a LangGraph-based AI application with frontend, backend, AI services, and supporting infrastructure.

## 🏗️ Architecture Overview

This project deploys a full-stack AI application built on LangGraph with the following components:

### Application Services
- **Frontend** (`gc-frontend`): React/Vite-based UI (Port 3000)
- **Backend** (`gc-backend`): API server (Port 8001)
- **AI Service** (`gc-ai`): LangGraph AI API (Port 8000)

### Infrastructure Services
- **PostgreSQL** (v16): Primary database with 10Gi storage
- **Redis** (v6): Caching and session storage with 5Gi storage
- **SeaweedFS**: Distributed file storage with 20Gi storage

### DevOps & Networking
- **ArgoCD**: GitOps deployment automation
- **Traefik**: Ingress controller and load balancer
- **Cert-manager**: Automatic SSL certificate management via Let's Encrypt
- **Helm Charts**: Package management and templating

## 🚀 Quick Start

### Prerequisites
- Kubernetes cluster (v1.20+)
- Helm 3.x installed
- kubectl configured
- ArgoCD installed in the cluster
- Traefik ingress controller
- Cert-manager configured


## ⚙️ Configuration

### Key Configuration Files

- [`charts/gc-app/values.yaml`](charts/gc-app/values.yaml): Main configuration values
- [`argocd-app.yaml`](argocd-app.yaml): ArgoCD application definition
- [`charts/gc-app/Chart.yaml`](charts/gc-app/Chart.yaml): Helm chart metadata

### Customizable Values

Modify [`values.yaml`](charts/gc-app/values.yaml) to customize:

#### Infrastructure Settings
```yaml
storageClass: "hcloud-volumes"  # Adjust for your cloud provider
```

#### Domain Configuration
```yaml
ingress:
  hosts:
    frontend: "k8s.idre.live"
    backend: "k8s.gcb.nikolanikolovski.com"
```

#### Resource Allocation
```yaml
# Adjust based on your workload
postgres:
  storage: "10Gi"
redis:
  storage: "5Gi"
seaweedfs:
  storage: "20Gi"
```

#### Application Scaling
```yaml
# Update replica counts in templates as needed
spec:
  replicas: 1  # Increase for high availability
```

## 🔐 Security & Secrets

The application requires several secrets to be pre-created:

1. **Image Pull Secret** (`regcred`): For pulling private container images
2. **Database Secret** (`postgres-secret`): PostgreSQL password
3. **Application Environment** (`app-env`): Backend and AI service environment variables



## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-feature`
3. Make changes and test thoroughly
4. Commit changes: `git commit -am 'Add new feature'`
5. Push to branch: `git push origin feature/new-feature`
6. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---
