# Kubernetes GitOps Platform with ArgoCD & Istio Service Mesh

This repository contains an infrastructure-as-code (GitOps) setup to provision ArgoCD, Istio Service Mesh, and a sample microservices-based e-commerce application on a standard Kubernetes cluster (like Minikube or Docker Desktop).

Everything is templated using **Helm** charts and GitOps principles.

## Repository Structure

- `argocd/`: Contains the "App of Apps" ArgoCD Application manifests that orchestrate the deployment of the cluster environment using sync waves.
  - **Wave 1:** `istio-base` (Istio Custom Resource Definitions)
  - **Wave 2:** `istiod` (Istio Control Plane)
  - **Wave 3:** `istio-ingress` (Istio Ingress Gateway)
  - **Wave 4:** `ecommerce` (The E-Commerce Application)
- `charts/ecommerce-app`: Helm chart deploying a dummy microservices-based e-commerce application integrated with the Service Mesh.

## Prerequisites

1. A Vanilla Kubernetes Cluster (Minikube or Docker Desktop).
2. `kubectl` CLI installed and authenticated to your cluster.
3. This repository pushed to your own Git hosting provider (e.g., GitHub).

## 🚀 Bootstrap Instructions

### 1. Start your Local Cluster
If you are using Minikube, start your cluster. We recommend allocating a decent amount of memory for Istio:
```bash
minikube start --memory=4096 --cpus=4
```

### 2. Update Repository URLs
Before applying the manifests, you must update the repository links in the ArgoCD applications to point to your fork/clone of this repository.

Replace `sumanthlagadapati` with your GitHub username in `argocd/app-of-apps.yaml` and `argocd/infra/ecommerce.yaml`.

*Note: The Istio manifests (`istio-base.yaml`, `istiod.yaml`, etc.) point directly to the official Istio Helm repositories, so they do not need to be changed.*

### 3. Install ArgoCD
ArgoCD must be installed first to bootstrap the remaining components. We will install it directly from the official manifests:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
*Wait a minute for ArgoCD pods to start up:*
```bash
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s
```

### 4. Deploy the App-of-Apps
Once ArgoCD is running, bootstrap the rest of the cluster by applying the root App-of-Apps manifest:

```bash
kubectl apply -f argocd/app-of-apps.yaml
```

### 5. Accessing ArgoCD and Viewing the Magic
To watch the deployment happen, you can port-forward the ArgoCD UI to your localhost:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
You can now access the UI at **https://localhost:8080**.

**To login**, the username is `admin`. Get the initial password by running:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Once logged in, you will see ArgoCD automatically deploying Istio and your E-Commerce application based on the Sync Waves!
