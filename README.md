CI/CD Deployment of a Python App on Kubernetes
Overview
This project demonstrates the end-to-end deployment pipeline of a containerized Python application using the following stack:
- CI: Azure DevOps Pipelines
- CD: Argo CD
- Container Registry: Azure Container Registry (ACR)
- Kubernetes Cluster: AKS or self-hosted
- Secrets Management: Azure Identity with Kubernetes Service Account
- IaC / GitOps: Declarative manifests managed via Git and synced by Argo CD
Tech Stack
Component           | Tool
--------------------|--------------------------
Source Code        | Python (Flask / FastAPI)
CI Pipeline        | Azure DevOps
CD Tool            | Argo CD
Container Registry | Azure Container Registry
K8s Cluster        | AKS / kubeadm cluster
Identity/Auth      | Workload Identity / Azure AD
Secrets Handling   | Azure Key Vault + Kubernetes SA
Monitoring         | Prometheus + Grafana (optional)
CI Pipeline (Azure DevOps)
Pipeline Tasks:
1. Trigger on commit to main branch
2. Run unit tests
3. Linting & static code analysis (e.g. flake8, pylint)
4. Build Docker image
5. Push to ACR
6. Update Helm chart/image tag (or trigger Argo CD sync if GitOps)



Sample YAML snippet:
trigger:
- main
pool:
  vmImage: 'zaki'
steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.10'
- script: |
    pip install -r requirements.txt
    pytest
  displayName: 'Run tests'
- task: Docker@2
  inputs:
    containerRegistry: 'MyACR'
    repository: 'my-python-app'
    command: 'buildAndPush'
    Dockerfile: '**/Dockerfile'
    tags: '$(Build.BuildId)'
CD Pipeline (Argo CD)
Deployment strategy: GitOps with Argo CD
App manifests stored in Git: either plain YAML or Helm charts
Argo CD Application CRD is defined and syncs with Git
Sample Application Manifest:
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: python-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/k8s-manifests
    targetRevision: HEAD
    path: apps/python-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
Application Deployment
- Python app is containerized using a Dockerfile
- Deployed to Kubernetes using Helm or Kustomize
- Used service account with proper Azure AD identity binding to access ACR and secrets
Identity & Secrets Management
- Enabled Workload Identity / Azure AD Pod Identity
- A Kubernetes ServiceAccount was annotated and linked with a Managed Identity
- Application pod retrieved secrets securely from Azure Key Vault via CSI Driver or SDK
ServiceAccount Sample:
apiVersion: v1
kind: ServiceAccount
metadata:
  name: python-app-sa
  annotations:
    azure.workload.identity/client-id: <client-id>
Monitoring & Observability
- Prometheus + Grafana stack deployed
- Configured custom metrics and alerts
- Pod logs available via kubectl logs or via EFK/PLG stack
Security Considerations
- Role-based access control (RBAC) scoped for Argo CD, DevOps pipeline, and app pods
- Secrets not stored in Git (used Key Vault)
- Image pulls are authorized using Azure identity (no static secrets)
Key Learnings and Achievements
- Mastered end-to-end CI/CD with a GitOps model
- Improved security posture by using identity-based access to ACR and secrets
- Automated application updates from code to cluster using DevOps best practices
