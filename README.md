# nginx-k8s
# Roadmap: Nginx on Kubernetes with GitHub Actions CI/CD

## 🗺️ Overview

```
Code Push → GitHub Actions → Build & Push Docker Image → Deploy to Kubernetes
```

---

## Phase 1: Prerequisites & Setup

**Tools to install locally:**
- `kubectl` – Kubernetes CLI
- `docker` – Container runtime
- A Kubernetes cluster (pick one):
  - **Local:** Minikube, Kind, or Docker Desktop
  - **Cloud:** GKE (Google), EKS (AWS), or AKS (Azure)
- A **Docker Hub** or **GitHub Container Registry (GHCR)** account

---

## Phase 2: Project Structure

```
nginx-k8s/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions pipeline
├── k8s/
│   ├── deployment.yaml         # Kubernetes Deployment
│   ├── service.yaml            # Kubernetes Service
│   └── ingress.yaml            # (Optional) Ingress
├── nginx/
│   └── nginx.conf              # Custom Nginx config (optional)
├── Dockerfile                  # Docker image definition
└── index.html                  # Your web content
```

---

## Phase 3: Dockerize Nginx

**`Dockerfile`**
```dockerfile
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```

**`index.html`** — your static content.

**Test locally:**
```bash
docker build -t my-nginx .
docker run -p 8080:80 my-nginx
```

---

## Phase 4: Kubernetes Manifests

### `k8s/deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: your-dockerhub-username/my-nginx:latest
          ports:
            - containerPort: 80
```

### `k8s/service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer   # Use NodePort for Minikube
```

**Apply manually (first time):**
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get pods
kubectl get svc
```

---

## Phase 5: GitHub Actions Pipeline

### GitHub Secrets to configure:
| Secret Name | Value |
|---|---|
| `DOCKER_USERNAME` | Your Docker Hub username |
| `DOCKER_PASSWORD` | Your Docker Hub password/token |
| `KUBE_CONFIG` | Base64-encoded `~/.kube/config` |

**Encode your kubeconfig:**
```bash
cat ~/.kube/config | base64
```

### `.github/workflows/deploy.yml`
```yaml
name: Deploy Nginx to Kubernetes

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # 1. Checkout code
      - name: Checkout Code
        uses: actions/checkout@v3

      # 2. Login to Docker Hub
      - name: Docker Login
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      # 3. Build & Push Docker Image
      - name: Build and Push Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/my-nginx:${{ github.sha }}

      # 4. Set up kubectl
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3

      # 5. Configure kubeconfig
      - name: Configure Kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > ~/.kube/config

      # 6. Update image in deployment & apply
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/nginx-deployment \
            nginx=${{ secrets.DOCKER_USERNAME }}/my-nginx:${{ github.sha }}
          kubectl rollout status deployment/nginx-deployment
```

---

## Phase 6: Full CI/CD Flow

```
1. Developer pushes to `main` branch
          ↓
2. GitHub Actions triggered
          ↓
3. Docker image built with commit SHA tag
          ↓
4. Image pushed to Docker Hub
          ↓
5. kubectl updates the Kubernetes Deployment
          ↓
6. Kubernetes performs a rolling update (zero downtime)
          ↓
7. New Nginx pods are live ✅
```

---

## Phase 7: Enhancements (Next Steps)

| Feature | How |
|---|---|
| 🔐 TLS/HTTPS | Add `cert-manager` + Ingress |
| 📊 Monitoring | Prometheus + Grafana |
| 🔁 Rollback | `kubectl rollout undo deployment/nginx-deployment` |
| 🌍 Custom Domain | Configure Ingress with your domain |
| 🧪 Staging Environment | Add a `staging` branch with separate namespace |
| 📦 Helm Charts | Package K8s manifests with Helm |

---

## Quick Start Checklist

- [ ] Install kubectl, docker, minikube
- [ ] Create Dockerfile + index.html
- [ ] Build & test Docker image locally
- [ ] Write K8s deployment + service YAML
- [ ] Apply manifests and verify pods run
- [ ] Push project to GitHub
- [ ] Add secrets to GitHub repo settings
- [ ] Create `.github/workflows/deploy.yml`
- [ ] Push a commit and watch the pipeline run!

---

Want me to dive deeper into any specific phase, or generate the complete set of files ready to use?
