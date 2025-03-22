# Kubernetes GitOps with ArgoCD

## Overview
This project demonstrates the implementation of GitOps using ArgoCD for managing Kubernetes deployments. GitOps ensures that the desired state of a Kubernetes cluster is maintained using version-controlled configurations.

## Features
- Declarative Infrastructure : Manage Kubernetes manifests via Git.
- Continuous Deployment : Automatically sync application states.
- Single cluster Management : Deploy and monitor application across single clusters.
- Automated Syncing : Ensure applications stay in their desired state without manual intervention.

## Prerequisites
- Kubernetes Cluster (Minikube, EKS, GKE, AKS, etc.)
- ArgoCD installed and configured
- Git repository with Kubernetes manifests
- kubectl installed
- argocd CLI installed
- Docker installed (if building container images)
---

## Step-by-Step Installation

### 1. Install ArgoCD in Kubernetes
```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Verify ArgoCD Installation
```sh
kubectl get pods -n argocd
```
Ensure all pods are in `Running` state:
```
NAME                                        READY   STATUS    RESTARTS   AGE
argocd-server-7f7b6dbd9b-xxxxx              1/1     Running   0          2m
argocd-repo-server-xxxx                     1/1     Running   0          2m
argocd-dex-server-xxxxx                     1/1     Running   0          2m
argocd-redis-xxxxx                          1/1     Running   0          2m
```

### 3. Expose ArgoCD Server (Port-forwarding or Ingress)
#### Option 1: Port Forwarding (Local Access)
```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Now, access `https://localhost:8080` in your browser.


Retrieve the external IP and access ArgoCD via `https://<EXTERNAL-IP>`.

#### Option 3: Using Ingress (Recommended for Production)
Apply an ingress resource pointing to the ArgoCD service.

---

### 4. Retrieve Initial Admin Password
```sh
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Use this password to log in to the UI or CLI.

### 5. Login to ArgoCD via CLI
```sh
argocd login <ARGOCD-SERVER> --username admin --password <retrieved-password>
```

### 6. Register a Git Repository with ArgoCD
```sh
argocd repo add <your-git-repo-url>
```
Ensure your repository contains Kubernetes manifests or Helm charts.

### 7. Deploy an Application with ArgoCD
```sh
argocd app create my-app \
    --repo <your-git-repo-url> \
    --path <k8s-manifest-path> \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace default
```

### 8. Sync and Monitor Application
```sh
argocd app sync my-app
argocd app get my-app
```
Check the status of the application:
```sh
argocd app list
```

---

## Automate GitOps Deployment
Enable auto-syncing to ensure ArgoCD automatically applies any changes in the Git repository:
```sh
argocd app set my-app --sync-policy automated
```

Enable pruning and self-healing to remove outdated resources:
```sh
argocd app set my-app --sync-policy automated --auto-prune --self-heal
```

---

## Running the Application
Once deployed, retrieve the service information:
```sh
kubectl get svc -n default
```
If using port-forwarding:
```sh
kubectl port-forward svc/my-app-service 8080:80 -n default
```
Now, open `http://localhost:8080` in your browser.

---

## Folder Structure
```
📦kubernetes-gitops
 ┣ 📂manifests
 ┃ ┣ 📜deployment.yaml
 ┃ ┣ 📜service.yaml
 ┃ ┗ 📜ingress.yaml
 ┣ 📂helm-chart
 ┃ ┣ 📜Chart.yaml
 ┃ ┗ 📜values.yaml
 ┗ 📜README.md
```

---

## Best Practices
- Keep application configurations in a separate Git repository.
- Set up automated sync policies for self-healing.
- Monitor application state with ArgoCD dashboards.
---

## Troubleshooting

### Check ArgoCD Logs
```sh
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server
```

### Restart ArgoCD Components
```sh
kubectl rollout restart deployment argocd-server -n argocd
```

### Force Resync Application
```sh
argocd app sync my-app --force
```

### Debugging Sync Failures
```sh
argocd app history my-app
argocd app diff my-app
```

---

## Scaling ArgoCD for Production
### Enable High Availability Mode
Deploy ArgoCD with HA mode for large-scale production environments:
```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml
```

### Monitor ArgoCD with Prometheus & Grafana
```sh
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/metrics.yaml
```

---

## References
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kubernetes GitOps](https://kubernetes.io/docs/concepts/gitops/)
---

## Author
Sapna Kamble
Syeda Maseera Tabassum
