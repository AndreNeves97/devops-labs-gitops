# ArgoCD Application Bootstrap Guide

## Prerequisites

1. **ArgoCD must be installed** in your Kubernetes cluster
2. **kubectl** must be configured to access your cluster
3. **ArgoCD namespace** (`argocd`) must exist in your cluster

## Step 1: Verify ArgoCD Installation

Check if ArgoCD is installed and running:

```bash
kubectl get pods -n argocd
```

If ArgoCD is not installed, install it first:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Step 2: Apply the ArgoCD Application Manifest

Apply the Application manifest to bootstrap your GitOps setup:

```bash
kubectl apply -f devops-labs/argocd/application.yml
```

## Step 3: Access the application

```bash
kubectl port-forward service/backend 3000:3000
kubectl port-forward service/frontend 8081:80
```

## Step 4: Verify Application Status

Check if the application was created:

```bash
kubectl get applications -n argocd
```

Check the detailed status:

```bash
kubectl describe application devops-labs -n argocd
```

## Step 5: Monitor Sync Status

Watch the application sync in real-time:

```bash
kubectl get application devops-labs -n argocd -w
```

Or access the ArgoCD UI:

```bash
# Port-forward to access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get the admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

# Access at https://localhost:8080
# Username: admin
# Password: (from command above)
```

## Step 5: Manual Sync (if needed)

If automatic sync doesn't happen immediately, you can manually trigger a sync:

```bash
kubectl patch application devops-labs -n argocd --type merge -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"HEAD"}}}'
```

Or use ArgoCD CLI:

```bash
argocd app sync devops-labs
```

## Troubleshooting

### Application shows "Unknown" or "Missing" status

1. Check if ArgoCD can access the Git repository:
   ```bash
   kubectl logs -n argocd -l app.kubernetes.io/name=argocd-repo-server
   ```

2. Verify repository credentials if it's private (you may need to configure repository secrets in ArgoCD)

### Sync fails

Check the application events:
```bash
kubectl describe application devops-labs -n argocd
```

Check ArgoCD application controller logs:
```bash
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
```

## What Happens Next?

Once the application is initialized:
1. ArgoCD will connect to your Git repository
2. It will read the manifests from `devops-labs/` path
3. It will apply all resources (frontend, backend, ingress, etc.) to the `default` namespace
4. With `automated: selfHeal: true`, ArgoCD will continuously monitor and auto-sync any drift

