
# 📘 ArgoCD: GitOps Continuous Delivery for Kubernetes

ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. It automates the deployment and lifecycle management of applications using Git repositories as the source of truth.

---

## 🚀 Key Features

- Declarative application management (stored in Git)
- Real-time application status monitoring
- Automated syncing and self-healing
- Support for Helm, Kustomize, plain YAML, and more
- RBAC and SSO integration
- Multi-cluster and multi-environment support
- Web UI, CLI, and REST API

---

## 🔧 Core Concepts

### 1. Application

The core unit in ArgoCD. An Application represents the desired state of a deployment defined in a Git repository.

Key fields:

- repoURL: Git repository URL
- targetRevision: Git branch, tag, or commit
- path: Path to manifests in the repo
- destination: Kubernetes cluster and namespace
- syncPolicy: Defines manual or automated sync behavior

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/myapp.git
    targetRevision: HEAD
    path: manifests/
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

### 2. Project

Projects are used to group applications and enforce boundaries (e.g., allowed repositories, clusters, or namespaces). Useful for multi-team or multi-tenant setups.

---

### 3. Sync

Sync aligns the live state of the application in the cluster with the desired state in Git.

- Manual Sync: Triggered by user via CLI or UI
- Auto Sync: ArgoCD automatically syncs when changes are detected in Git

Sync options:

- Prune: Delete resources not in Git
- SelfHeal: Revert out-of-band changes automatically

---

### 4. Health Status

Indicates whether deployed resources are functioning correctly.

Common statuses:

- Healthy
- Degraded
- Progressing
- Missing
- Unknown
- Suspended

---

### 5. Application Status

- Synced: Cluster state matches Git
- OutOfSync: Detected drift from Git
- Syncing: Currently applying changes

---

## 🏗️ Architecture Components

- argocd-server: REST API and Web UI
- argocd-repo-server: Fetches Git repositories and renders manifests
- argocd-application-controller: Reconciles applications
- argocd-dex-server: (optional) Handles SSO (LDAP, OIDC, etc.)

---

## 🛠️ Installation

Install ArgoCD in a Kubernetes cluster:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access ArgoCD UI:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

---

## 📦 Creating an Application

Via CLI:

```bash
argocd app create myapp \
  --repo https://github.com/my-org/myapp.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Sync manually:

```bash
argocd app sync myapp
```

Enable auto-sync:

```bash
argocd app set myapp --sync-policy automated
```

---

## 🧩 ApplicationSet (Multi-Cluster/Env Deployments)

ApplicationSets allow you to generate multiple applications from a single template using generators.

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: apps-by-cluster
spec:
  generators:
  - clusters: {}
  template:
    metadata:
      name: '{{name}}-app'
    spec:
      project: default
      source:
        repoURL: https://github.com/my-org/myapp.git
        path: manifests/
        targetRevision: HEAD
      destination:
        server: '{{server}}'
        namespace: default
```

---

## 🔐 Security Best Practices

- Enable SSO (OIDC, LDAP)
- Restrict access using RBAC
- Store secrets securely (sealed secrets or external secret stores)
- Limit access to the ArgoCD API/UI via NetworkPolicies or firewalls
- Enable audit logs for traceability

---

## 📈 Monitoring & Troubleshooting

- Web UI and CLI (argocd app get <app>)
- Logs:
  - argocd-server
  - argocd-repo-server
  - argocd-application-controller

---

## 📚 Useful Commands

Login:

```bash
argocd login localhost:8080
```

List apps:
```bash
argocd app list
```
App details:
```bash
argocd app get myapp
```
Delete app:
```bash
argocd app delete myapp
```
---

from pathlib import Path

# Define the Terraform markdown content
terraform_md_content = """
# Terraform Cheat Sheet & Case Study

## What is Terraform?
Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp. It enables you to define and provision infrastructure using a declarative configuration language (HCL - HashiCorp Configuration Language).

## Basic Terraform Workflow
1. Write .tf configuration files.
2. Run `terraform init` to initialize the project.
3. Run `terraform plan` to preview changes.
4. Run `terraform apply` to apply changes.
5. Run `terraform destroy` to delete the infrastructure.

## Key Concepts
- Providers: Interface between Terraform and cloud platforms (e.g., AWS, Azure).
- Resources: Describe the infrastructure objects (e.g., EC2 instances, S3 buckets).
- Variables: Inputs to parameterize configurations.
- Outputs: Export information from your configuration.
- Modules: Reusable group of resources.
- State: Keeps track of resources managed by Terraform.

## Common Commands
```bash
terraform init      # Initialize Terraform configuration
terraform plan      # Preview changes
terraform apply     # Apply changes
terraform destroy   # Destroy managed infrastructure
terraform validate  # Validate configuration
terraform fmt       # Format .tf files
terraform output    # View output values
