# ArgoCD Multi-Cluster Deployment Architecture
## Comprehensive Guide for Client Demo

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [ArgoCD Overview](#argocd-overview)
3. [Architecture Overview](#architecture-overview)
4. [Multi-Cluster Setup](#multi-cluster-setup)
5. [Helm Charts Integration](#helm-charts-integration)
6. [CI/CD Workflow](#cicd-workflow)
7. [Project Structure](#project-structure)
8. [Deployment Strategy](#deployment-strategy)
9. [Best Practices](#best-practices)
10. [Demo Walkthrough](#demo-walkthrough)

---

## Executive Summary

This document presents a comprehensive architecture for **ArgoCD Multi-Cluster Deployment** using **Helm Charts** for GitOps-based continuous delivery. The solution enables automated, declarative deployments across multiple Kubernetes clusters with environment-specific configurations.

### Key Benefits
- **GitOps Approach**: Single source of truth in Git repository
- **Multi-Environment Support**: Staging and Production environments
- **Automated Deployments**: ArgoCD continuously syncs desired state
- **Helm Chart Templating**: Reusable, parameterized Kubernetes manifests
- **CI/CD Integration**: Jenkins pipeline for automated image updates
- **Declarative Configuration**: Infrastructure as Code (IaC)

---

## ArgoCD Overview

### What is ArgoCD?

**ArgoCD** is a declarative, GitOps continuous delivery tool for Kubernetes. It follows the GitOps pattern where:

- **Git Repository** = Source of Truth
- **ArgoCD** = Continuous Sync Engine
- **Kubernetes Clusters** = Target Environments

### Core Concepts

#### 1. **Application**
An ArgoCD Application represents a deployment of a complete application to a Kubernetes cluster. It defines:
- Source repository (Git)
- Target cluster
- Sync policy
- Health checks

#### 2. **Sync Policy**
- **Manual Sync**: User-triggered deployments
- **Auto Sync**: Automatic synchronization when Git changes
- **Sync Options**: Prune, Self-heal, Allow empty

#### 3. **Health Status**
ArgoCD monitors application health:
- **Healthy**: All resources are in desired state
- **Degraded**: Some resources are not healthy
- **Progressing**: Deployment in progress
- **Suspended**: Application sync is paused
- **Unknown**: Health status cannot be determined

#### 4. **Multi-Cluster Support**
ArgoCD can manage multiple Kubernetes clusters:
- **Hub Cluster**: Where ArgoCD is installed
- **Remote Clusters**: Target clusters for deployments
- **Cluster Credentials**: Stored securely in ArgoCD

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CI/CD Pipeline                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                 │
│  │ Jenkins  │───▶│  Build   │───▶│   Push   │                 │
│  │ Pipeline │    │  Image   │    │  to Git  │                 │
│  └──────────┘    └──────────┘    └──────────┘                 │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Git Repository (GitHub)                     │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  • Helm Charts (prayag-app1/)                             │  │
│  │  • Environment Values (staging/prod)                      │  │
│  │  • Kubernetes Manifests                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ArgoCD (Hub Cluster)                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  • Application Definitions                                │  │
│  │  • Sync Policies                                         │  │
│  │  • Multi-Cluster Management                              │  │
│  │  • Health Monitoring                                     │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────┬───────────────────────────────┬─────────────────────────┘
        │                               │
        ▼                               ▼
┌──────────────────┐          ┌──────────────────┐
│  Staging Cluster │          │ Production Cluster│
│  (kind/local)    │          │  (kind/local)    │
│                  │          │                  │
│  ┌────────────┐  │          │  ┌────────────┐  │
│  │ Deployment│  │          │  │ Deployment│  │
│  │ Service    │  │          │  │ Service    │  │
│  │ Pods (x2)  │  │          │  │ Pods (x2)  │  │
│  └────────────┘  │          │  └────────────┘  │
└──────────────────┘          └──────────────────┘
```

### Component Details

#### 1. **CI/CD Pipeline (Jenkins)**
- **Purpose**: Build Docker images and update Git manifests
- **Workflow**:
  1. Checkout code from Git
  2. Build Docker image
  3. Update deployment manifests with new image tag
  4. Commit and push changes to Git

#### 2. **Git Repository (Source of Truth)**
- **Helm Charts**: Templated Kubernetes manifests
- **Values Files**: Environment-specific configurations
- **Manifests**: Direct Kubernetes YAML files

#### 3. **ArgoCD (GitOps Controller)**
- **Monitors**: Git repository for changes
- **Syncs**: Desired state to target clusters
- **Manages**: Multiple clusters and applications
- **Reports**: Application health and sync status

#### 4. **Kubernetes Clusters**
- **Staging**: Development and testing environment
- **Production**: Live production environment
- **Deployments**: Managed by ArgoCD

---

## Multi-Cluster Setup

### Cluster Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    ArgoCD Hub Cluster                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  ArgoCD Components:                                   │  │
│  │  • argocd-server (UI & API)                           │  │
│  │  • argocd-application-controller                      │  │
│  │  • argocd-repo-server                                 │  │
│  │  • argocd-dex-server (Authentication)                 │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   Cluster 1  │   │   Cluster 2  │   │   Cluster N  │
│  (Staging)   │   │ (Production) │   │  (Optional)  │
│              │   │              │   │              │
│  • Apps      │   │  • Apps      │   │  • Apps      │
│  • Services  │   │  • Services  │   │  • Services  │
│  • Ingress   │   │  • Ingress   │   │  • Ingress   │
└──────────────┘   └──────────────┘   └──────────────┘
```

### Cluster Registration Process

1. **Install ArgoCD on Hub Cluster**
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. **Register Remote Clusters**
   - Get cluster credentials
   - Add cluster to ArgoCD:
     ```bash
     argocd cluster add <context-name>
     ```

3. **Verify Cluster Connection**
   - Check cluster status in ArgoCD UI
   - Test deployment to remote cluster

### Cluster Configuration

Each cluster is configured with:
- **Cluster Name**: Unique identifier
- **Server URL**: Kubernetes API endpoint
- **Credentials**: Service account or kubeconfig
- **Labels**: Environment tags (staging, prod)

---

## Helm Charts Integration

### What are Helm Charts?

**Helm** is the package manager for Kubernetes. Helm Charts provide:
- **Templating**: Parameterized Kubernetes manifests
- **Reusability**: Single chart for multiple environments
- **Versioning**: Chart versioning and dependency management
- **Values Override**: Environment-specific configurations

### Project Helm Chart Structure

```
helm-charts/
├── prayag-app1/                    # Application Helm Chart
│   ├── Chart.yaml                  # Chart metadata
│   ├── values.yaml                 # Default values
│   ├── templates/                  # Kubernetes manifest templates
│   │   ├── _helpers.tpl           # Template helpers
│   │   ├── deployment.yaml        # Deployment template
│   │   └── service.yaml           # Service template
│   └── env/                        # Environment-specific values
│       ├── values-staging.yaml    # Staging overrides
│       └── values-prod.yaml       # Production overrides
└── env/                            # Global environment values
    ├── values-staging.yaml
    └── values-prod.yaml
```

### Chart Components

#### 1. **Chart.yaml**
```yaml
apiVersion: v2
name: prayag-new1
description: Helm chart for prayag-new1 application
type: application
version: 0.1.0
appVersion: "1.0.0"
```

#### 2. **values.yaml** (Default)
```yaml
replicaCount: 4

image:
  repository: prayag8tiwari/prayag-new1-pipeline
  tag: "1.0.0-7"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8080

resources:
  limits:
    memory: "256Mi"
    cpu: "500m"
  requests:
    memory: "128Mi"
    cpu: "250m"

containerPort: 8080
```

#### 3. **Environment-Specific Values**

**values-staging.yaml:**
```yaml
replicaCount: 2
image:
  tag: "1.0.0-staging"
resources:
  limits:
    cpu: 300m
    memory: 256Mi
env: "staging"
```

**values-prod.yaml:**
```yaml
replicaCount: 2
image:
  tag: "1.0.0-prod"
resources:
  limits:
    cpu: 300m
    memory: 256Mi
env: "production"
```

### Helm Template Example

**deployment.yaml template:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "prayag-new1.fullname" . }}
  labels:
    app: {{ include "prayag-new1.name" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ include "prayag-new1.name" . }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        resources:
{{ toYaml .Values.resources | indent 12 }}
```

### ArgoCD Helm Integration

ArgoCD supports Helm charts in two ways:

#### 1. **Helm Chart from Git Repository**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: prayag-app1-staging
spec:
  source:
    repoURL: https://github.com/prayag-gwl/ArgoCD-Multi-Cluster-Deployment.git
    path: helm-charts/prayag-app1
    helm:
      valueFiles:
      - env/values-staging.yaml
  destination:
    server: https://staging-cluster.example.com
    namespace: default
```

#### 2. **Helm Chart from Helm Repository**
```yaml
spec:
  source:
    repoURL: https://charts.example.com
    chart: prayag-app1
    targetRevision: 0.1.0
    helm:
      values: |
        replicaCount: 2
        image:
          tag: "1.0.0-staging"
```

### Benefits of Helm + ArgoCD

1. **Environment Parity**: Same chart, different values
2. **Configuration Management**: Centralized and versioned
3. **Rollback Capability**: Helm history + ArgoCD sync
4. **Template Reusability**: DRY principle
5. **Dependency Management**: Chart dependencies

---

## CI/CD Workflow

### Complete Workflow Diagram

```
Developer                    Jenkins                    Git                    ArgoCD              Kubernetes
   │                          │                         │                       │                    │
   │  Push Code               │                         │                       │                    │
   ├─────────────────────────▶│                         │                       │                    │
   │                          │                         │                       │                    │
   │                          │  Trigger Pipeline       │                       │                    │
   │                          │                         │                       │                    │
   │                          │  Checkout Code          │                       │                    │
   │                          ├────────────────────────▶│                       │                    │
   │                          │                         │                       │                    │
   │                          │  Build Docker Image     │                       │                    │
   │                          │  (Tag: IMAGE_TAG)       │                       │                    │
   │                          │                         │                       │                    │
   │                          │  Update deployment.yaml │                       │                    │
   │                          │  with IMAGE_TAG         │                       │                    │
   │                          │                         │                       │                    │
   │                          │  Commit & Push          │                       │                    │
   │                          ├────────────────────────▶│                       │                    │
   │                          │                         │                       │                    │
   │                          │                         │  Git Change Detected  │                    │
   │                          │                         ├──────────────────────▶│                    │
   │                          │                         │                       │                    │
   │                          │                         │                       │  Sync Application  │
   │                          │                         │                       ├───────────────────▶│
   │                          │                         │                       │                    │
   │                          │                         │                       │  Deploy to Cluster │
   │                          │                         │                       │                    │
   │                          │                         │                       │  Health Check      │
   │                          │                         │                       │◀───────────────────│
   │                          │                         │                       │                    │
   │                          │                         │                       │  Status: Healthy   │
   │                          │                         │                       │                    │
```

### Jenkins Pipeline Stages

#### Stage 1: Cleanup Workspace
```groovy
stage("Cleanup Workspace") {
    steps {
        cleanWs()
    }
}
```

#### Stage 2: Checkout from SCM
```groovy
stage("Checkout from SCM") {
    steps {
        git branch: 'main', 
            credentialsId: 'github', 
            url: 'https://github.com/prayag-gwl/ArgoCD-Multi-Cluster-Deployment.git'
    }
}
```

#### Stage 3: Update Deployment Tags
```groovy
stage("Update the Deployment Tags") {
    steps {
        sh """
            sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
        """
    }
}
```

#### Stage 4: Push to Git
```groovy
stage("Push the changed deployment file to Git") {
    steps {
        sh """
            git config --global user.name "User"
            git config --global user.email "user@example.com"
            git add deployment.yaml
            git commit -m "Updated Deployment Manifest"
        """
        withCredentials([gitUsernamePassword(credentialsId: 'github')]) {
            sh "git push origin main"
        }
    }
}
```

### ArgoCD Auto-Sync

Once Git is updated, ArgoCD:
1. **Detects Change**: Polls Git repository (or webhook)
2. **Compares State**: Desired (Git) vs Actual (Cluster)
3. **Syncs Application**: Applies changes to cluster
4. **Health Check**: Monitors application health
5. **Reports Status**: Updates UI with sync status

---

## Project Structure

### Complete Directory Structure

```
ArgoCD-Multi-Cluster-Deployment/
│
├── argocd-applications.yaml                  # ArgoCD Application definitions (Deploy this)
├── ArgoCD-Multi-Cluster-Architecture-Documentation.md  # This document
├── DEPLOYMENT_GUIDE.md                       # Deployment instructions
├── DEPLOYMENT_SUMMARY.md                     # Quick deployment reference
│
├── helm-charts/                              # Helm Charts Directory
│   ├── prayag-app1/                          # Application Helm Chart
│   │   ├── Chart.yaml                        # Chart metadata
│   │   ├── values.yaml                       # Default values
│   │   ├── templates/                        # Kubernetes templates
│   │   │   ├── _helpers.tpl                 # Template helpers
│   │   │   ├── deployment.yaml              # Deployment template
│   │   │   ├── service.yaml                 # Service template
│   │   │   └── ingress.yaml                 # Ingress template (for HTTPS)
│   │   └── env/                             # Environment values
│   │       ├── values-staging.yaml          # Staging overrides (NodePort:8090)
│   │       └── values-prod.yaml             # Production overrides (NodePort:8090)
│   └── env/                                  # Global environment values
│       ├── values-staging.yaml
│       └── values-prod.yaml
│
├── deployment.yaml                           # Direct Kubernetes deployment
├── service.yaml                              # Direct Kubernetes service
├── Jenkinsfile                               # Jenkins CI/CD pipeline
└── README.md                                 # Project README
```

### File Purposes

| File/Directory | Purpose |
|---------------|---------|
| `argocd-applications.yaml` | **ArgoCD Application definitions - Deploy this file to create applications** |
| `DEPLOYMENT_GUIDE.md` | Complete deployment instructions and troubleshooting |
| `DEPLOYMENT_SUMMARY.md` | Quick reference for deployment |
| `helm-charts/prayag-app1/` | Main application Helm chart |
| `helm-charts/prayag-app1/Chart.yaml` | Chart metadata and version |
| `helm-charts/prayag-app1/values.yaml` | Default configuration values |
| `helm-charts/prayag-app1/templates/` | Kubernetes manifest templates |
| `helm-charts/prayag-app1/templates/ingress.yaml` | Ingress template for HTTPS access |
| `helm-charts/prayag-app1/env/values-*.yaml` | Environment-specific overrides (NodePort, Ingress config) |
| `deployment.yaml` | Direct Kubernetes deployment (non-Helm) |
| `service.yaml` | Direct Kubernetes service (non-Helm) |
| `Jenkinsfile` | CI/CD pipeline definition |

---

## Deployment Strategy

### Environment Strategy

#### Staging Environment
- **Purpose**: Development and testing
- **Replicas**: 2
- **Image Tag**: `latest`
- **Service Type**: NodePort on port 8090
- **Access**: `http://localhost:8090` (via port-forward)
- **Resources**: Lower limits (300m CPU, 256Mi memory)
- **Ingress**: Enabled (ready for HTTPS)
- **Auto-Sync**: Enabled for rapid iteration

#### Production Environment
- **Purpose**: Live production workloads
- **Replicas**: 2 (configurable)
- **Image Tag**: `latest`
- **Service Type**: NodePort on port 8090
- **Access**: `http://localhost:8090` (via port-forward)
- **Resources**: Optimized limits (300m CPU, 256Mi memory)
- **Ingress**: Enabled (ready for HTTPS)
- **Auto-Sync**: Manual sync recommended for safety

### Deployment Methods

#### Method 1: ArgoCD Applications (Recommended)

**Deploy using the provided argocd-applications.yaml file:**

```bash
# Deploy both staging and production applications
kubectl apply -f argocd-applications.yaml

# Verify applications are created
kubectl get applications -n argocd

# Check application status
argocd app list
argocd app get prayag-app1-staging
argocd app get prayag-app1-production
```

The `argocd-applications.yaml` file contains ready-to-use Application definitions that:
- Point to your Git repository: `https://github.com/prayag-gwl/ArgoCD-Multi-Cluster-Deployment.git`
- Use Helm charts from `helm-charts/prayag-app1`
- Reference environment-specific values files
- Configure auto-sync for staging, manual sync for production

**Alternative: Install using Helm directly**

```bash
helm install prayag-app1-staging ./helm-charts/prayag-app1 \
  -f ./helm-charts/prayag-app1/env/values-staging.yaml \
  -n staging
```

#### Method 2: Direct Kubernetes Manifests
```bash
# Apply directly
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Or via ArgoCD
kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: prayag-app1-direct
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/prayag-gwl/ArgoCD-Multi-Cluster-Deployment.git
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: default
EOF
```

### Sync Policies

#### Automated Sync (Staging)
```yaml
syncPolicy:
  automated:
    prune: true        # Delete resources not in Git
    selfHeal: true     # Auto-sync if cluster drifts
    allowEmpty: false  # Fail if no resources
```

#### Manual Sync (Production)
```yaml
syncPolicy:
  syncOptions:
  - CreateNamespace=true
  # No automated section = manual sync
```

### Rollback Strategy

1. **Helm Rollback**
   ```bash
   helm rollback prayag-app1-staging <revision-number>
   ```

2. **ArgoCD Rollback**
   - Use ArgoCD UI: Application → History → Rollback
   - Or via CLI: `argocd app rollback <app-name>`

3. **Git Revert**
   - Revert commit in Git
   - ArgoCD will auto-sync (if enabled)

---

## Best Practices

### 1. GitOps Best Practices

#### ✅ DO:
- **Single Source of Truth**: All manifests in Git
- **Immutable Infrastructure**: Never modify cluster directly
- **Version Control**: Tag releases and chart versions
- **Pull Requests**: Review changes before merge
- **Branch Strategy**: Use branches for environments

#### ❌ DON'T:
- Modify resources directly in cluster
- Store secrets in Git (use sealed-secrets or external secrets)
- Commit sensitive data
- Bypass Git for deployments

### 2. Helm Chart Best Practices

#### ✅ DO:
- **Use Values Files**: Environment-specific configurations
- **Template Functions**: Use Helm template functions
- **Chart Versioning**: Semantic versioning (1.0.0)
- **Documentation**: Document all values
- **Testing**: Test charts before deployment

#### ❌ DON'T:
- Hardcode values in templates
- Mix environments in single values file
- Skip chart versioning
- Ignore linting errors

### 3. ArgoCD Best Practices

#### ✅ DO:
- **Application per Environment**: Separate apps for staging/prod
- **Namespace Isolation**: Use namespaces for environments
- **RBAC**: Implement role-based access control
- **Health Checks**: Configure proper health checks
- **Sync Windows**: Use sync windows for production

#### ❌ DON'T:
- Use same app for multiple environments
- Skip health checks
- Allow unrestricted access
- Ignore sync status

### 4. Multi-Cluster Best Practices

#### ✅ DO:
- **Cluster Labels**: Label clusters by environment
- **Centralized Management**: Hub cluster for ArgoCD
- **Network Policies**: Secure cluster communication
- **Monitoring**: Monitor all clusters
- **Backup**: Backup cluster configurations

#### ❌ DON'T:
- Expose cluster credentials
- Skip network security
- Ignore cluster health
- Deploy without testing

### 5. CI/CD Best Practices

#### ✅ DO:
- **Automated Testing**: Test before deployment
- **Image Tagging**: Use semantic versioning
- **Git Commits**: Meaningful commit messages
- **Pipeline Stages**: Clear stage separation
- **Error Handling**: Proper error handling

#### ❌ DON'T:
- Deploy untested code
- Use `latest` tag in production
- Skip validation
- Ignore pipeline failures

---

## Demo Walkthrough

### Prerequisites

1. **Kubernetes Clusters**
   - Hub cluster (ArgoCD installed)
   - Staging cluster
   - Production cluster

2. **Tools Installed**
   - `kubectl`
   - `helm`
   - `argocd` CLI
   - `docker` (for image building)

3. **Access**
   - Git repository access
   - Cluster credentials
   - ArgoCD admin credentials

### Demo Steps

#### Step 1: Setup ArgoCD (if not already done)

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**Access**: https://localhost:8080 (admin / <password>)

#### Step 2: Register Clusters

```bash
# Login to ArgoCD
argocd login localhost:8080

# Add staging cluster
argocd cluster add <staging-context> --name staging

# Add production cluster
argocd cluster add <prod-context> --name production

# Verify clusters
argocd cluster list
```

#### Step 3: Deploy ArgoCD Applications

**Deploy using the provided argocd-applications.yaml file:**

```bash
# Deploy both staging and production applications
kubectl apply -f argocd-applications.yaml

# Verify applications are created
kubectl get applications -n argocd
```

The `argocd-applications.yaml` file contains:
- **Staging Application**: Auto-sync enabled, uses `env/values-staging.yaml`
- **Production Application**: Manual sync, uses `env/values-prod.yaml`

Both applications:
- Point to: `https://github.com/prayag-gwl/ArgoCD-Multi-Cluster-Deployment.git`
- Use Helm chart: `helm-charts/prayag-app1`
- Configured for NodePort service on port 8090
- Include Ingress configuration for HTTPS support

#### Step 4: Verify Deployment

```bash
# Check application status
argocd app get prayag-app1-staging
argocd app get prayag-app1-production

# View in UI
# Navigate to Applications → prayag-app1-staging
```

#### Step 5: Demonstrate GitOps Flow

1. **Make a Change**
   ```bash
   # Update values-staging.yaml
   # Change replicaCount from 2 to 3
   ```

2. **Commit and Push**
   ```bash
   git add helm-charts/prayag-app1/env/values-staging.yaml
   git commit -m "Scale staging to 3 replicas"
   git push origin main
   ```

3. **Watch ArgoCD Sync**
   - Open ArgoCD UI
   - Watch application sync automatically
   - Verify replicas scaled to 3

#### Step 6: Access Application at localhost:8090

After deployment, access the application:

```bash
# Port forward the service to localhost:8090
kubectl port-forward svc/prayag-app1-staging-prayag-new1 -n staging 8090:8080

# For production
kubectl port-forward svc/prayag-app1-production-prayag-new1 -n production 8090:8080
```

**Access the application:**
- **Staging**: `http://localhost:8090`
- **Production**: `http://localhost:8090` (stop staging port-forward first, or use different port)

**Verify it's working:**
```bash
curl http://localhost:8090
```

**Note**: For HTTPS access at `https://localhost:8090`, you'll need to:
1. Enable Ingress (already configured in values files)
2. Install Ingress Controller (e.g., NGINX)
3. Configure TLS certificates
4. Port-forward the Ingress controller

#### Step 7: Demonstrate Rollback

```bash
# Option 1: Rollback via ArgoCD
argocd app rollback prayag-app1-staging <revision>

# Option 2: Revert in Git
git revert HEAD
git push origin main
# ArgoCD will auto-sync the revert
```

#### Step 8: Show Multi-Cluster View

- **ArgoCD UI**: Show both applications
- **Cluster Comparison**: Compare staging vs production
- **Health Status**: Show health indicators
- **Sync Status**: Show sync states

### Demo Highlights

1. **GitOps Workflow**: Show Git → ArgoCD → Cluster flow
2. **Multi-Environment**: Demonstrate staging and production
3. **Helm Integration**: Show Helm chart templating
4. **Auto-Sync**: Demonstrate automatic synchronization
5. **Application Access**: Show application running at `http://localhost:8090`
6. **Health Monitoring**: Show health checks
7. **Rollback**: Demonstrate rollback capabilities

---

## Troubleshooting

### Common Issues

#### 1. Application Out of Sync
```bash
# Check sync status
argocd app get <app-name>

# Force sync
argocd app sync <app-name>

# Check logs
argocd app logs <app-name>
```

#### 2. Helm Chart Errors
```bash
# Validate chart
helm lint ./helm-charts/prayag-app1

# Dry run
helm install <release> ./helm-charts/prayag-app1 --dry-run --debug
```

#### 3. Cluster Connection Issues
```bash
# Test cluster connection
kubectl --context=<cluster-context> get nodes

# Re-add cluster to ArgoCD
argocd cluster remove <cluster-name>
argocd cluster add <cluster-context> --name <cluster-name>
```

#### 4. Sync Failures
```bash
# Check application controller logs
kubectl logs -n argocd deployment/argocd-application-controller

# Check repo server logs
kubectl logs -n argocd deployment/argocd-repo-server
```

---

## Security Considerations

### 1. Access Control
- **RBAC**: Implement role-based access control
- **SSO Integration**: Use SSO for authentication
- **Cluster Credentials**: Secure cluster credentials

### 2. Secrets Management
- **External Secrets**: Use external-secrets-operator
- **Sealed Secrets**: Use sealed-secrets
- **Vault Integration**: Integrate with HashiCorp Vault

### 3. Network Security
- **Network Policies**: Implement network policies
- **TLS**: Use TLS for all connections
- **Firewall Rules**: Restrict cluster access

### 4. Git Security
- **Branch Protection**: Protect main branch
- **Code Review**: Require PR reviews
- **Secrets Scanning**: Scan for secrets in Git

---

## Monitoring and Observability

### Key Metrics to Monitor

1. **Application Health**
   - Sync status
   - Health status
   - Resource availability

2. **Cluster Health**
   - Node status
   - Resource usage
   - Network connectivity

3. **ArgoCD Metrics**
   - Sync duration
   - Error rates
   - API performance

### Recommended Tools

- **Prometheus**: Metrics collection
- **Grafana**: Visualization and dashboards
- **AlertManager**: Alerting
- **Loki**: Log aggregation

---

## Conclusion

This architecture provides a robust, scalable solution for multi-cluster deployments using ArgoCD and Helm charts. Key benefits include:

- ✅ **GitOps**: Single source of truth in Git
- ✅ **Automation**: Automated deployments and sync
- ✅ **Multi-Environment**: Support for staging and production
- ✅ **Helm Integration**: Reusable, templated charts
- ✅ **Scalability**: Support for multiple clusters
- ✅ **Observability**: Health monitoring and status reporting

### Next Steps

1. **Expand Environments**: Add development, QA environments
2. **Advanced Features**: Implement sync windows, hooks
3. **Monitoring**: Set up comprehensive monitoring
4. **Security**: Enhance security with RBAC, secrets management
5. **CI/CD Integration**: Enhance Jenkins pipeline

---

## References

### External Resources
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [GitOps Principles](https://www.gitops.tech/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

### Project Documentation
- **DEPLOYMENT_GUIDE.md**: Complete deployment instructions and troubleshooting
- **DEPLOYMENT_SUMMARY.md**: Quick reference for deployment
- **argocd-applications.yaml**: Ready-to-use ArgoCD Application definitions
- **Git Repository**: [https://github.com/prayag-gwl/ArgoCD-Multi-Cluster-Deployment](https://github.com/prayag-gwl/ArgoCD-Multi-Cluster-Deployment)

---

**Document Version**: 1.1  
**Last Updated**: 2025-12-02  
**Repository**: https://github.com/prayag-gwl/ArgoCD-Multi-Cluster-Deployment  
**Author**: Architecture Team

