# 🚀 GitOps Production Platform with ArgoCD

A production-grade GitOps delivery platform built on Kubernetes, featuring automated CI/CD with GitHub Actions and GitOps-based deployments via ArgoCD across multiple environments.

---

## 📐 Architecture

```
Developer pushes code
         │
         ▼
┌─────────────────┐
│  GitHub Actions  │  ← Triggered on push to master
│  CI Pipeline     │
└────────┬────────┘
         │
    ┌────┴─────┐
    │          │
    ▼          ▼
┌────────┐  ┌──────────────┐
│Docker  │  │ GitOps-config│  ← Updates image tag
│  Hub   │  │    Repo      │
└────────┘  └──────┬───────┘
                   │
                   ▼ (watches every 2 min)
          ┌────────────────┐
          │    ArgoCD      │
          └───────┬────────┘
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
  ┌─────────┐ ┌─────────┐ ┌─────────┐
  │   dev   │ │ staging │ │  prod   │
  │ auto ✅ │ │ auto ✅ │ │ manual🔒│
  │ 1 rep   │ │ 2 reps  │ │ 3 reps  │
  └─────────┘ └─────────┘ └─────────┘
```

---

## 🔄 GitOps Flow

```
git push → GitHub Actions → Build & Push Image → Update GitOps-config → ArgoCD deploys
```

1. Developer pushes code to `GitOps-app` repo
2. GitHub Actions CI pipeline triggers automatically
3. Docker image is built and pushed to Docker Hub with commit SHA as tag
4. Pipeline updates `deployment.yaml` in `GitOps-config` repo with new image tag
5. ArgoCD detects the change and deploys to `dev` and `staging` automatically
6. `prod` deployment requires manual approval via ArgoCD UI

---

## 🏗️ Project Structure

```
GitOps-config/
├── apps/
│   ├── base/
│   │   ├── deployment.yaml      # Base Kubernetes manifest
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   ├── dev/
│   │   └── kustomization.yaml   # 1 replica, auto-sync
│   ├── staging/
│   │   └── kustomization.yaml   # 2 replicas, auto-sync
│   └── prod/
│       └── kustomization.yaml   # 3 replicas, manual gate
└── argocd/
    ├── app-dev.yaml
    ├── app-staging.yaml
    └── app-prod.yaml

GitOps-app/
├── app.py                       # Flask application
├── Dockerfile
├── requirements.txt
└── .github/
    └── workflows/
        └── ci.yaml              # GitHub Actions pipeline
```

---

## ⚙️ Tech Stack

| Tool | Purpose |
|------|---------|
| **ArgoCD** | GitOps continuous delivery |
| **Kubernetes (Minikube)** | Container orchestration |
| **Kustomize** | Multi-environment configuration |
| **GitHub Actions** | CI pipeline automation |
| **Docker** | Containerization |
| **Docker Hub** | Container registry |
| **Flask** | Sample application |

---

## 🌍 Environments

| Environment | Sync Policy | Replicas | Use Case |
|-------------|-------------|----------|----------|
| `dev` | Automatic ✅ | 1 | Development testing |
| `staging` | Automatic ✅ | 2 | Pre-production validation |
| `prod` | Manual 🔒 | 3 | Production release gate |

---

## 🚀 Setup & Installation

### Prerequisites
- Minikube
- kubectl
- Helm
- Docker

### 1. Start Minikube Cluster
```bash
minikube start --nodes 2 --cpus 2 --memory 4096 --profile gitops-cluster
```

### 2. Install ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3. Access ArgoCD UI
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Get password
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

### 4. Deploy ArgoCD Applications
```bash
kubectl apply -f argocd/
```

### 5. Configure GitHub Actions Secrets
| Secret | Description |
|--------|-------------|
| `DOCKERHUB_USERNAME` | Docker Hub username |
| `DOCKERHUB_PASSWORD` | Docker Hub access token |
| `GITHUB_ACCESS_TOKEN` | GitHub PAT with repo + workflow scopes |

---

## 📸 Screenshots

### ArgoCD Dashboard — All Environments
> dev and staging auto-synced, prod awaiting manual approval

---

## 🎯 Key Features

- **Zero-touch deployments** — push code and it deploys automatically
- **Multi-environment promotion** — dev → staging → prod with manual gate
- **Image immutability** — every build tagged with unique commit SHA
- **Self-healing** — ArgoCD reverts manual cluster changes automatically
- **Separation of concerns** — app code and config in separate repos

---

## 📝 Author

**Amr Adel** — Junior DevOps Engineer  
