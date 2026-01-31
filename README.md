# 📦 TodoAssistant Helm Chart

Kubernetes Helm chart for deploying the TodoAssistant application with GitOps and CI/CD automation.

## 🌟 Overview

This repository contains the Helm chart for deploying the **TodoAssistant** application to Kubernetes clusters. It is designed to work seamlessly with ArgoCD for GitOps-based continuous deployment and is integrated with the CI/CD pipeline from the main TodoAssistant repository.

### Key Features

- **Production-Ready Helm Chart**: Complete Kubernetes manifests for TodoAssistant deployment
- **GitOps Integration**: Designed for ArgoCD synchronization and automated deployments
- **Dynamic Tag Updates**: CI/CD pipeline automatically updates image tags
- **Multi-Environment Support**: Easily configurable for dev, staging, and production
- **Scalable Architecture**: Kubernetes-native design for horizontal scaling
- **Security Best Practices**: RBAC, network policies, and secret management

## 🏗️ Repository Structure

```
TODOASSISTANT-HELM/
├── todo-summary-assistant/          # Main Helm chart directory
│   ├── Chart.yaml                   # Chart metadata and version
│   ├── values.yaml                  # Default configuration values
│   ├── templates/                   # Kubernetes manifest templates
│   │   ├── backend-deployment.yaml  # Backend service deployment
│   │   ├── backend-service.yaml     # Backend service definition
│   │   ├── frontend-deployment.yaml # Frontend service deployment
│   │   ├── frontend-ingress.yaml    # Frontend ingress configuration
│   │   ├── frontend-service.yaml    # Frontend service definition
│   │   ├── mysql-headless-service.yaml  # MySQL headless service
│   │   ├── mysql-pv.yaml            # MySQL persistent volume
│   │   ├── mysql-pvc.yaml           # MySQL persistent volume claim
│   │   ├── mysql-secret.yaml        # MySQL credentials secret
│   │   ├── mysql-statefulset.yaml   # MySQL StatefulSet
│   │   ├── namespace.yaml           # Kubernetes namespace
│   │   └── serviceaccount.yaml      # Service account
│   └── README.md                    # Chart documentation
└── README.md                        # This file
```

### Component Breakdown

**Backend Components:**
- `backend-deployment.yaml` - Spring Boot API deployment configuration
- `backend-service.yaml` - Backend service exposure

**Frontend Components:**
- `frontend-deployment.yaml` - React application deployment
- `frontend-service.yaml` - Frontend service definition
- `frontend-ingress.yaml` - Ingress rules for external access

**Database Components:**
- `mysql-statefulset.yaml` - MySQL StatefulSet for persistent database
- `mysql-headless-service.yaml` - Headless service for StatefulSet pods
- `mysql-pv.yaml` - Persistent Volume for database storage
- `mysql-pvc.yaml` - Persistent Volume Claim
- `mysql-secret.yaml` - Database credentials and configuration

**Infrastructure:**
- `namespace.yaml` - Isolated Kubernetes namespace
- `serviceaccount.yaml` - Service account for pod identity

## 🔄 GitOps Workflow

This Helm chart is part of an automated GitOps workflow:

```
CI/CD Pipeline → Build & Scan Images → Push to Registry → Update values.yaml 
→ Git Commit → ArgoCD Detects Changes → Sync to Rancher Cluster → Deployment
```

### How It Works

1. **CI/CD Trigger**: Jenkins pipeline in the main TodoAssistant repo builds new Docker images
2. **Image Scanning**: Trivy scans images for security vulnerabilities
3. **Registry Push**: Validated images are pushed to DockerHub with version tags
4. **Dynamic Tag Update**: Jenkins automatically updates `values.yaml` with new image tags
5. **Git Commit**: Changes are committed to this repository
6. **ArgoCD Detection**: ArgoCD monitors this repo and detects the tag update
7. **Automatic Sync**: ArgoCD synchronizes the changes to the Rancher Kubernetes cluster
8. **Rolling Update**: Kubernetes performs a rolling update with zero downtime

## 🚀 Quick Start

### Prerequisites

- **Kubernetes Cluster**: Version 1.20+ (Rancher Desktop, Minikube, EKS, GKE, AKS, etc.)
- **Helm**: Version 3.0 or higher
- **kubectl**: Configured to access your cluster
- **ArgoCD**: (Optional) For GitOps-based deployments

### Installation

#### Method 1: Direct Helm Install

```bash
# Add or clone the repository
git clone https://github.com/Dilip-Devopos/TodoAssistant-helm.git
cd TodoAssistant-helm

# Create namespace
kubectl create namespace todo-app

# Install the chart
helm install todo-assistant ./todo-summary-assistant \
  --namespace todo-app \
  --set image.tag=latest

# Verify deployment
kubectl get pods -n todo-app
kubectl get services -n todo-app
```

#### Method 2: ArgoCD GitOps Deployment

```bash
# Create ArgoCD Application
kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: todo-assistant
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/Dilip-Devopos/TodoAssistant-helm.git
    targetRevision: main
    path: todo-summary-assistant
  destination:
    server: https://kubernetes.default.svc
    namespace: todo-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
EOF

# Monitor sync status
argocd app get todo-assistant
```

## ⚙️ Configuration

### Key Configuration Options

The `values.yaml` file contains all configurable options. Here are the most important ones:

#### Image Configuration

```yaml
image:
  repository: dockerhub-username/todo-assistant
  tag: "v1.0.0"                    # Dynamically updated by CI/CD
  pullPolicy: IfNotPresent
```

#### Replica Configuration

```yaml
replicaCount: 3                     # Number of pod replicas
```

#### Resource Limits

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

#### Service Configuration

```yaml
service:
  type: ClusterIP                   # ClusterIP, NodePort, or LoadBalancer
  port: 8080
  targetPort: 8080
```

#### Ingress Configuration

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: todo-app.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: todo-app-tls
      hosts:
        - todo-app.example.com
```

#### Database Configuration

```yaml
database:
  host: mysql-service
  port: 3306
  name: todoapp
  username: todouser
  # Password should be stored in Kubernetes Secret
```

#### Autoscaling Configuration

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

### Customizing Values

You can override default values in multiple ways:

**Command Line:**
```bash
helm install todo-assistant ./todo-summary-assistant \
  --namespace todo-app \
  --set image.tag=v2.0.0 \
  --set replicaCount=5 \
  --set service.type=LoadBalancer
```

**Custom Values File:**
```bash
# Create custom-values.yaml
cat > custom-values.yaml <<EOF
image:
  tag: v2.0.0
replicaCount: 5
service:
  type: LoadBalancer
ingress:
  enabled: true
  hosts:
    - host: my-todo-app.example.com
EOF

# Install with custom values
helm install todo-assistant ./todo-summary-assistant \
  --namespace todo-app \
  --values custom-values.yaml
```

## 🔐 Security Considerations

### Secrets Management

Never commit sensitive data to Git. Use Kubernetes Secrets or external secret management:

**Create Database Secret:**
```bash
kubectl create secret generic todo-db-credentials \
  --from-literal=password=your-secure-password \
  --namespace todo-app
```

**Reference in values.yaml:**
```yaml
database:
  existingSecret: todo-db-credentials
  passwordKey: password
```

### RBAC Configuration

The chart includes ServiceAccount and RBAC resources:

```yaml
serviceAccount:
  create: true
  name: todo-assistant-sa
  annotations: {}

rbac:
  create: true
```

### Network Policies

Enable network policies to restrict traffic:

```yaml
networkPolicy:
  enabled: true
  policyTypes:
    - Ingress
    - Egress
```

## 📊 Monitoring and Observability

### Health Checks

The chart includes liveness and readiness probes:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

### Metrics and Monitoring

Expose metrics for Prometheus:

```yaml
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
    namespace: monitoring
```

## 🔧 Operations

### Upgrade Deployment

```bash
# Upgrade with new values
helm upgrade todo-assistant ./todo-summary-assistant \
  --namespace todo-app \
  --set image.tag=v2.0.0

# Check upgrade status
helm status todo-assistant -n todo-app

# View release history
helm history todo-assistant -n todo-app
```

### Rollback Deployment

```bash
# Rollback to previous version
helm rollback todo-assistant -n todo-app

# Rollback to specific revision
helm rollback todo-assistant 3 -n todo-app
```

### Uninstall

```bash
# Uninstall the release
helm uninstall todo-assistant -n todo-app

# Delete namespace (optional)
kubectl delete namespace todo-app
```

## 🔍 Troubleshooting

### Common Issues

**Pods not starting:**
```bash
# Check pod status
kubectl get pods -n todo-app

# View pod logs
kubectl logs <pod-name> -n todo-app

# Describe pod for events
kubectl describe pod <pod-name> -n todo-app
```

**Image pull errors:**
```bash
# Verify image exists in registry
docker pull <image-repository>:<tag>

# Check image pull secret
kubectl get secrets -n todo-app
kubectl describe secret <image-pull-secret> -n todo-app
```

**Service not accessible:**
```bash
# Check service endpoints
kubectl get endpoints -n todo-app

# Test service connectivity
kubectl run test-pod --image=busybox -i --tty --rm -- wget -O- http://todo-assistant:8080/health
```

**Helm chart validation:**
```bash
# Lint the chart
helm lint ./todo-summary-assistant

# Dry-run installation
helm install todo-assistant ./todo-summary-assistant \
  --namespace todo-app \
  --dry-run --debug

# Template validation
helm template todo-assistant ./todo-summary-assistant \
  --namespace todo-app
```

### ArgoCD Sync Issues

```bash
# Check ArgoCD application status
argocd app get todo-assistant

# Force sync
argocd app sync todo-assistant

# View sync logs
argocd app logs todo-assistant
```

## 📝 CI/CD Integration

### Automated Tag Updates

The CI/CD pipeline automatically updates image tags in `values.yaml`:

**Jenkins Pipeline Step:**
```groovy
stage('Update Helm Chart') {
    steps {
        script {
            // Update values.yaml with new image tag
            sh """
                sed -i 's|tag: .*|tag: "${NEW_TAG}"|g' todo-summary-assistant/values.yaml
                git add todo-summary-assistant/values.yaml
                git commit -m "Update image tag to ${NEW_TAG}"
                git push origin main
            """
        }
    }
}
```

This triggers ArgoCD to detect changes and deploy automatically.

## 🌐 Multi-Environment Deployments

### Environment-Specific Values

Create separate values files for each environment:

**values-dev.yaml:**
```yaml
replicaCount: 1
resources:
  limits:
    cpu: 200m
    memory: 256Mi
ingress:
  hosts:
    - host: todo-dev.example.com
```

**values-prod.yaml:**
```yaml
replicaCount: 5
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
ingress:
  hosts:
    - host: todo.example.com
```

**Deploy to specific environment:**
```bash
# Development
helm install todo-assistant-dev ./todo-summary-assistant \
  --namespace todo-dev \
  --values values-dev.yaml

# Production
helm install todo-assistant-prod ./todo-summary-assistant \
  --namespace todo-prod \
  --values values-prod.yaml
```

## 🤝 Contributing

Contributions to improve the Helm chart are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improve-chart`)
3. Make your changes
4. Test the chart (`helm lint`, `helm template`)
5. Commit your changes (`git commit -m 'Add new feature'`)
6. Push to the branch (`git push origin feature/improve-chart`)
7. Open a Pull Request

## 📖 Additional Resources

- **Main Application Repository**: [TodoAssistant](https://github.com/Dilip-Devopos/TodoAssistant)
- **Helm Documentation**: https://helm.sh/docs/
- **ArgoCD Documentation**: https://argo-cd.readthedocs.io/
- **Kubernetes Documentation**: https://kubernetes.io/docs/

## 📄 Chart Information

- **Chart Name**: todo-summary-assistant
- **Chart Type**: Application
- **Kubernetes Version**: >= 1.20
- **Maintainer**: Dilip-Devopos

## 🎯 Best Practices

1. **Version Control**: Always commit value changes through CI/CD pipeline
2. **Secret Management**: Never commit secrets to Git; use Kubernetes Secrets or external vaults
3. **Resource Limits**: Always set resource requests and limits for production
4. **Health Checks**: Configure proper liveness and readiness probes
5. **Monitoring**: Enable metrics and integrate with monitoring solutions
6. **Backup**: Regularly backup your Helm releases and configurations
7. **Testing**: Test chart changes in development before deploying to production
8. **Documentation**: Keep this README updated with any chart modifications

## 🔄 Release Management

### Versioning

The chart follows [Semantic Versioning](https://semver.org/):
- **MAJOR**: Incompatible API changes
- **MINOR**: Backwards-compatible functionality additions
- **PATCH**: Backwards-compatible bug fixes

### Creating Releases

```bash
# Update Chart.yaml version
# Update CHANGELOG.md

# Package the chart
helm package ./todo-summary-assistant

# Publish to chart repository (if applicable)
helm repo index .
```

---

**Deployed with ♥️ using GitOps and Kubernetes**
