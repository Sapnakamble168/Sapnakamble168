# Kubernetes GitOps with ArgoCD

## Overview
This project demonstrates the implementation of **GitOps** using **ArgoCD** for managing Kubernetes deployments.
GitOps ensures that the desired state of a Kubernetes cluster is maintained using version-controlled configurations.

## Features
- Declarative Infrastructure : Manage Kubernetes manifests via Git.
- Continuous Deployment : Automatically sync application states.
- Rollback & Self-healing : Revert changes easily and maintain cluster integrity.
- Visibility & Auditing : Track all deployment changes.

## Prerequisites
- Kubernetes Cluster (Minikube, EKS, GKE, AKS, etc.)
- ArgoCD installed and configured
- Git repository with Kubernetes manifests
- Helm (if using Helm charts)
- kubectl installed
- argocd CLI installed

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
Ensure all pods are in `Running` state.

### 3. Access ArgoCD UI
```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Open `https://localhost:8080` in your browser.

### 4. Retrieve Initial Admin Password
```sh
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 5. Login to ArgoCD via CLI
```sh
argocd login localhost:8080 --username admin --password <retrieved-password>
```

### 6. Register a Git Repository
```sh
argocd repo add <your-git-repo-url>
```

### 7. Deploy an Application
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

### 9. Automate Sync for Continuous Deployment
Enable auto-syncing to ensure ArgoCD automatically applies any changes in the Git repository:
```sh
argocd app set my-app --sync-policy automated
```
## Running the Application
Once your application is deployed and synchronized, access it using:
```sh
kubectl get svc -n default
```
If using port-forwarding:
```sh
kubectl port-forward svc/my-app-service 8080:80 -n default
```
Now, open `http://localhost:8080` in your browser.

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

## Best Practices
- Keep application configurations in a separate Git repository.
- Enable RBAC and authentication in ArgoCD.
- Use Helm or Kustomize for better templating.
- Set up automated sync policies for self-healing.
- Monitor application state with ArgoCD dashboards.
- Set resource limits in Kubernetes manifests to avoid overuse.

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

## References
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kubernetes GitOps](https://kubernetes.io/docs/concepts/gitops/)

## Author
Sapna Kamble
Syeda Maseera Tabassum
