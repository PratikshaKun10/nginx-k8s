# 🚀 Nginx on Kubernetes with GitHub Actions CI/CD

## 🗺️ Project Overview

This project demonstrates how to:

* Containerize an **Nginx application using Docker**
* Deploy it to **Kubernetes**
* Expose it using a **Kubernetes Service**
* Automate deployment using **GitHub Actions CI/CD**

### CI/CD Workflow

```
Developer Push Code
        │
        ▼
GitHub Actions Pipeline Triggered
        │
        ▼
Build Docker Image
        │
        ▼
Push Image to Docker Hub
        │
        ▼
Update Kubernetes Deployment
        │
        ▼
Rolling Update of Pods
        │
        ▼
Application Available via Kubernetes Service
```

---

# 🛠️ Phase 1: Prerequisites

Install the following tools:

| Tool       | Purpose                  |
| ---------- | ------------------------ |
| Docker     | Build container images   |
| kubectl    | Kubernetes CLI           |
| Minikube   | Local Kubernetes cluster |
| GitHub     | Source code management   |
| Docker Hub | Container registry       |

Verify installation:

```bash
docker --version
kubectl version --client
minikube version
```

---

# 📁 Phase 2: Project Structure

```
nginx-k8s/
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
├── nginx/
│   └── nginx.conf
│
├── Dockerfile
├── index.html
└── README.md
```

---

# 🐳 Phase 3: Dockerize Nginx

### Dockerfile

```dockerfile
FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx","-g","daemon off;"]
```

---

### index.html

Example webpage:

```html
<h1>Hello from Kubernetes 🚀</h1>
<p>Deployed using Nginx on Kubernetes</p>
```

---

### Build Docker Image

```bash
docker build -t my-nginx .
```

Run locally:

```bash
docker run -p 8080:80 my-nginx
```

Open in browser:

```
http://localhost:8080
```

---

# ☸️ Phase 4: Kubernetes Deployment

### deployment.yaml

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

---

# 🌐 Kubernetes Service

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: NodePort
```

Apply manifests:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Verify resources:

```bash
kubectl get pods
kubectl get svc
```

---

# 🔎 Understanding Kubernetes Workflow

```
User Request
     │
     ▼
Service (nginx-service)
     │
     ▼
Endpoints
     │
     ▼
Pods (Nginx Containers)
     │
     ▼
Deployment
```

---

# 📌 Check Service Endpoints

Command:

```bash
kubectl get endpoints nginx-service
```

Example output:

```
10.244.0.6:80
10.244.0.7:80
```

Explanation:

Endpoints represent the **actual Pod IP addresses** where the service forwards traffic.

---

# 🌍 Accessing the Application

### Method 1: Minikube Service

```bash
minikube service nginx-service
```

Example output:

```
http://192.168.49.2:32606
```

Minikube automatically creates a tunnel and opens the service in the browser.

---

### Method 2: Port Forward

```bash
kubectl port-forward svc/nginx-service 8080:80
```

Access application:

```
http://localhost:8080
```

---

# ⚙️ Phase 5: GitHub Actions CI/CD

Pipeline triggers on push to **main branch**.

Workflow steps:

1. Checkout code
2. Login to Docker Hub
3. Build Docker image
4. Push image to Docker Hub
5. Update Kubernetes deployment
6. Perform rolling update

---

# 🔐 GitHub Secrets Required

Configure the following secrets in GitHub:

| Secret          | Description               |
| --------------- | ------------------------- |
| DOCKER_USERNAME | Docker Hub username       |
| DOCKER_PASSWORD | Docker Hub token          |
| KUBE_CONFIG     | Base64 encoded kubeconfig |

Encode kubeconfig:

```bash
cat ~/.kube/config | base64
```

---

# 🔁 Rolling Update in Kubernetes

When a new image is deployed:

```
Old Pod Terminated
New Pod Created
Traffic Shifted Automatically
```

Verify rollout:

```bash
kubectl rollout status deployment/nginx-deployment
```

Rollback if needed:

```bash
kubectl rollout undo deployment/nginx-deployment
```
---

✅ This README now clearly shows:

* Kubernetes **Deployment**
* **Service**
* **Endpoints**
* **Minikube testing**
* **CI/CD pipeline**
