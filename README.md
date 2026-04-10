# OpenShift GitOps & Service Mesh Platform

This repository contains an infrastructure-as-code (GitOps) setup to provision Red Hat OpenShift GitOps (ArgoCD), OpenShift Service Mesh (Istio), and a sample microservices-based e-commerce application.

Everything is templated using **Helm** and tailored for **OpenShift 4.14**.

## Repository Structure

- `charts/platform-operators`: Helm chart to install essential operators (GitOps, Service Mesh, Elasticsearch, Jaeger, Kiali).
- `charts/service-mesh-control-plane`: Helm chart to instantiate the Istio Control Plane (SMCP) and Member Roll (SMMR).
- `charts/ecommerce-app`: Helm chart deploying a sample microservices-based e-commerce application integrated with the Service Mesh.
- `argocd/`: Contains the "App of Apps" ArgoCD Application manifests that orchestrate the deployment of the above charts using sync waves.

## Prerequisites

1. An OpenShift 4.14 Cluster.
2. `oc` or `kubectl` CLI installed and authenticated to your cluster with cluster-admin privileges.
3. This repository pushed to your own Git hosting provider (e.g., GitHub).

## 🚀 Bootstrap Instructions

### 1. Update Repository URLs
Before applying the manifests, you must update the repository links in the ArgoCD applications to point to your fork/clone of this repository.

Replace `YOUR_GIT_USERNAME` in all files under the `argocd/` directory with your actual Git username or the git URL of your repository:
- `argocd/app-of-apps.yaml`
- `argocd/infra/operators.yaml`
- `argocd/infra/smcp.yaml`
- `argocd/infra/ecommerce.yaml`

### 2. Install OpenShift GitOps (First-time bootstrap)
If OpenShift GitOps is not yet installed on your cluster, you first need to install the operator. You can apply the operator subscription directly from the Helm chart templates or wait for OLM to provision it:

```bash
oc apply -f charts/platform-operators/templates/subscriptions.yaml
```
*Wait a couple of minutes for the `openshift-gitops-operator` to install and the `openshift-gitops` namespace / ArgoCD instance to become available.*

### 3. Deploy the App-of-Apps
Once ArgoCD is running, bootstrap the rest of the cluster by applying the root App-of-Apps manifest:

```bash
oc apply -f argocd/app-of-apps.yaml
```

### 4. Watch the Magic Happen!
ArgoCD will now sync and cascade the deployment based on **Sync Waves**:
- **Wave 1**: Ensures all remaining requested operators (Elasticsearch, Jaeger, Kiali, Service Mesh) are installed and available.
- **Wave 2**: Deploys the Service Mesh Control Plane (`istio-system` namespace).
- **Wave 3**: Deploys the E-Commerce sample application (`ecommerce` namespace) and enrolls it into the mesh.

You can log in to the OpenShift GitOps (ArgoCD) UI (found in the OpenShift Console under the 9-box menu in the top right) to visualize the sync process! When the e-commerce app deploys, Kiali will begin showing the traffic topology graph between the frontend, product catalog, and cart services.
