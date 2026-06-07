
# Retail 3-Tier Microservices Platform (Local GitOps Setup)

This repository contains the GitOps infrastructure manifests for deploying a production-grade 3-tier retail microservices platform simulation locally inside a **Kind (Kubernetes in Docker)** cluster using **ArgoCD**. 

The architecture consists of a public-facing React frontend, a Node.js backend data API layer, and a pre-seeded PostgreSQL core database instance.

---

## 🏗️ Architecture Blueprint

The platform is optimized to run 100% offline and cost-free on local development environments by dynamically toggling cloud-native constraints (like AWS RDS strict SSL validation rules) using specialized local profiles.

* **Frontend UI:** React Application (Listens on internal port `80`)
* **Backend API:** Node.js Express Application (Listens on internal port `8080`)
* **Database Engine:** PostgreSQL 15 Alpine (Listens on internal port `5432`)

---

## 🛠️ Prerequisites

Ensure you have the following tools installed on your local development machine before initializing the bootstrapper:

* **Docker Desktop** (with WSL2 backend enabled for Windows 11)
* **Git Bash** or a comparable Linux/Unix terminal terminal matrix
* **Kind CLI** (`kubectl` management handles integrated)
* **Helm CLI** (v3+)

---

## 🚀 Step-by-Step Deployment Guide

### Step 1: Initialize the Kind Local Cluster Node
Create a Kind cluster configuration file mapping local management ports `80` and `443` into your local loopback infrastructure grid. Save this as `kind-config.yaml`:

```yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
name: retail-local
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP

```

Launch the cluster inside your Git Bash terminal:

```bash
kind create cluster --config kind-config.yaml

```

---

### Step 2: Install ArgoCD Control Planes

Deploy the ArgoCD control operator components using the official Helm package repository:

```bash
# Create the isolated control plane namespace
kubectl create namespace argocd

# Add and download the Helm package metadata tracks
helm repo add argo [https://argoproj.github.io/argo-helm](https://argoproj.github.io/argo-helm)
helm repo update

# Deploy ArgoCD via Helm
helm install argocd argo/argo-cd \
  --namespace argocd \
  --set server.service.type=ClusterIP

```

Extract and decode your auto-generated admin user password to access the UI interface:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode; echo

```

To log into the dashboard dashboard, open a tunnel proxy window:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443

```

👉 Access the panel UI dashboard at: **`https://localhost:8080`** (User: `admin`).

---

### Step 3: Sideload your Public Docker Hub Images

Since the application manifests target public registries, ensure your local development node tracks the exact public endpoints cleanly without authentication handshakes:

```bash
# Pull the latest image assets locally onto your system engine
docker pull docker.io/Chinthaparthy-UmasankarReddy/retail-backend:latest
docker pull docker.io/Chinthaparthy-UmasankarReddy/retail-frontend:latest

# Sideload the images straight into the running Kind cluster node cache
kind load docker-image docker.io/Chinthaparthy-UmasankarReddy/retail-backend:latest --name retail-local
kind load docker-image docker.io/Chinthaparthy-UmasankarReddy/retail-frontend:latest --name retail-local

```

---

### Step 4: Register the GitOps Application Root

Apply the primary declarative application orchestration manifest to link your repository branch tracking loops straight to the cluster tracking runtime state engines.

Create an **`application.yaml`** configuration:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: retail-3tier-local
  namespace: argocd
spec:
  project: default
  source:
    repoURL: '[https://github.com/Chinthaparthy-UmasankarReddy/retail-infra-gitops.git](https://github.com/Chinthaparthy-UmasankarReddy/retail-infra-gitops.git)'
    targetRevision: main
    path: manifests
  destination:
    server: '[https://kubernetes.default.svc](https://kubernetes.default.svc)'
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true

```

Register the application config via the CLI:

```bash
kubectl apply -f application.yaml

```

---

### Step 5: Establish Network Port-Forward Proxy Routes

Once ArgoCD sets up the stack and seeds the PostgreSQL tables natively using the mounted `init.sql` initialization scripts, spin up two distinct terminal proxies to interact with the platform:

* **Terminal 1 (Frontend Storefront Portal Routing):**
```bash
kubectl port-forward svc/retail-frontend-service 3000:80 -n production

```


* **Terminal 2 (Backend Core API Routing):**
```bash
kubectl port-forward svc/retail-backend-service 5000:8080 -n production

```



👉 Open up your web browser and load: **`http://localhost:3000`**

---

## 🔧 Local Configuration Optimization Details

### 1. Database Connection Management (`NODE_ENV=local`)

The application codebase contains an active profile block that dynamically strips away hardcoded AWS RDS strict SSL validation constraints when running locally inside Kind:

```javascript
if (process.env.NODE_ENV === 'local' || process.env.PGSSLMODE === 'disable') {
  console.log("Local development profile active: Disabling SSL connection parameters.");
  poolConfig.ssl = false; 
}

```

### 2. Native Auto-Seeding Workflow

Database tables and demo catalog matrix fields are auto-populated during the initial initialization phase using native PostgreSQL execution endpoints (`/docker-entrypoint-initdb.d`), managed through an internal Kubernetes ConfigMap inside `postgres-local.yaml`.

```

```