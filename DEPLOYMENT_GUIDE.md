# Deployment Guide - ArgoCD Multi-Cluster Deployment

## Overview

This guide explains how to deploy `prayag-app1` to both **staging** and **production** environments using ArgoCD, and access the application at `http://localhost:8090/`.

## Prerequisites

- ArgoCD installed and running on your cluster
- `kubectl` configured to access your cluster
- Git repository accessible by ArgoCD
- Helm chart ready in the repository

## Deployment Steps

### Step 1: Deploy ArgoCD Applications

Apply the ArgoCD Application manifests to create both staging and production applications:

```bash
kubectl apply -f argocd-applications.yaml
```

This will create two applications:
- `prayag-app1-staging` (auto-sync enabled)
- `prayag-app1-production` (manual sync)

### Step 2: Verify Applications in ArgoCD UI

1. Access ArgoCD UI (usually after port-forward):
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   Then open: https://localhost:8080

2. Login with admin credentials:
   ```bash
   # Get admin password
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
   ```

3. You should see both applications in the UI:
   - `prayag-app1-staging`
   - `prayag-app1-production`

### Step 3: Check Application Status

Using ArgoCD CLI:
```bash
# List applications
argocd app list

# Check staging application
argocd app get prayag-app1-staging

# Check production application
argocd app get prayag-app1-production
```

Using kubectl:
```bash
# Check applications
kubectl get applications -n argocd

# Check deployments
kubectl get deployments -n staging
kubectl get deployments -n production

# Check services
kubectl get svc -n staging
kubectl get svc -n production
```

### Step 4: Sync Applications

**Staging** (auto-sync enabled):
- Will sync automatically when Git changes
- Or manually: `argocd app sync prayag-app1-staging`

**Production** (manual sync):
```bash
argocd app sync prayag-app1-production
```

Or use the ArgoCD UI:
1. Select the application
2. Click "Sync" button
3. Confirm the sync

### Step 5: Access Application at localhost:8090

The application is configured with **NodePort** service on port **8090**. Access it using:

#### Option 1: Direct NodePort Access (if using local cluster like kind/minikube)

**For HTTP access:**
```bash
# Get the node IP (for kind/minikube)
kubectl get nodes -o wide

# Access via node IP
curl http://<NODE-IP>:8090
# or
curl http://localhost:8090
```

**For kind cluster:**
```bash
# Port forward from node port
docker port <kind-cluster-node> | grep 8090

# Or create port mapping
kubectl port-forward svc/<service-name> -n staging 8090:8080
```

#### Option 2: Port Forward (Recommended for local access)

```bash
# For staging
kubectl port-forward svc/prayag-app1-staging-prayag-new1 -n staging 8090:8080

# For production
kubectl port-forward svc/prayag-app1-production-prayag-new1 -n production 8090:8080
```

Then access: **http://localhost:8090**

#### Option 3: Using Ingress (if Ingress Controller is installed)

The Helm chart includes Ingress configuration. To use it:

1. Install Ingress Controller (if not already installed):
   ```bash
   # For NGINX Ingress
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
   ```

2. Update values files to enable ingress (already configured in env/values-*.yaml)

3. Access via Ingress:
   ```bash
   # Get ingress address
   kubectl get ingress -n staging
   
   # Access
   curl http://localhost/
   ```

4. Port forward for localhost access:
   ```bash
   kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8090:80
   ```

### Step 6: Verify Application is Running

```bash
# Check pods
kubectl get pods -n staging
kubectl get pods -n production

# Check service endpoints
kubectl get endpoints -n staging
kubectl get endpoints -n production

# Test the application
curl http://localhost:8090
```

## Configuration Details

### Staging Environment
- **Namespace**: `staging`
- **Replicas**: 2
- **Service Type**: NodePort (port 8090)
- **Image**: `prayag8tiwari/prayag-new1-pipeline:latest`
- **Auto-sync**: Enabled

### Production Environment
- **Namespace**: `production`
- **Replicas**: 2
- **Service Type**: NodePort (port 8090)
- **Image**: `prayag8tiwari/prayag-new1-pipeline:latest`
- **Auto-sync**: Manual (disabled for safety)

## Troubleshooting

### Application Not Syncing

```bash
# Check application status
argocd app get <app-name>

# Check logs
argocd app logs <app-name>

# Force refresh
argocd app get <app-name> --refresh

# Check Git repository connectivity
argocd repo list
```

### Service Not Accessible

```bash
# Check service
kubectl get svc -n <namespace>

# Check endpoints
kubectl get endpoints -n <namespace>

# Check pods
kubectl get pods -n <namespace>

# Check pod logs
kubectl logs <pod-name> -n <namespace>
```

### Port Already in Use

If port 8090 is already in use:

1. Change the NodePort in values files:
   ```yaml
   service:
     type: NodePort
     port: 8080
     nodePort: 8091  # Use different port
   ```

2. Commit and push changes to Git

3. ArgoCD will auto-sync (staging) or manually sync (production)

### Cannot Access via localhost

If you can't access via localhost:

1. **For kind/minikube**: Use node IP instead
   ```bash
   minikube service <service-name> -n <namespace> --url
   # or
   kind get nodes
   docker inspect <node-name> | grep IPAddress
   ```

2. **Port forward alternative**:
   ```bash
   kubectl port-forward svc/<service-name> -n <namespace> <local-port>:8080
   ```

## Next Steps

1. **Monitor Applications**: Use ArgoCD UI to monitor application health
2. **Update Configuration**: Make changes in Git, ArgoCD will sync automatically
3. **Rollback**: Use ArgoCD UI or CLI to rollback if needed
4. **Scaling**: Update replica count in values files and commit to Git

## Useful Commands

```bash
# Application Management
argocd app list
argocd app get <app-name>
argocd app sync <app-name>
argocd app rollback <app-name>

# Resource Management
kubectl get all -n staging
kubectl get all -n production

# Logs
kubectl logs -l app=prayag-new1 -n staging
kubectl logs -l app=prayag-new1 -n production

# Port Forward
kubectl port-forward svc/<service-name> -n <namespace> 8090:8080
```

## Notes

- **HTTPS Access**: For HTTPS at localhost:8090, you'll need to:
  1. Set up TLS certificates
  2. Use Ingress with TLS configuration
  3. Configure a reverse proxy (nginx/traefik)
  
- **Current Setup**: The current configuration provides HTTP access via NodePort. For HTTPS, consider using Ingress with proper TLS certificates.

- **Multiple Environments**: Both staging and production use the same port (8090) but in different namespaces. Use port-forward with different local ports if you need to access both simultaneously.

