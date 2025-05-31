
# 🎓 ArgoCD Final Exam Questions & Answers


## ✅ Easy Level

### 1. What is ArgoCD and what problem does it solve?
**Answer:** ArgoCD is a declarative GitOps continuous delivery tool for Kubernetes. It solves the problem of deploying and managing applications using Git as the single source of truth. It ensures cluster state matches Git and offers automated syncing, rollback, and monitoring.

### 2. What is GitOps? How does ArgoCD implement GitOps principles?
**Answer:** GitOps is a practice of using Git repositories as the source of truth for infrastructure and application deployment. ArgoCD implements GitOps by watching Git repositories, detecting changes, and applying them to Kubernetes clusters automatically or manually.

### 3. Name the main components of an ArgoCD Application manifest.
**Answer:**
- `apiVersion`, `kind`, `metadata`
- `spec`:
  - `project`
  - `source` (repoURL, path, targetRevision)
  - `destination` (cluster, namespace)
  - `syncPolicy` (manual or automated)

### 4. How does ArgoCD determine if an application is ‘Synced’ or ‘OutOfSync’?
**Answer:** ArgoCD compares the live state in the cluster with the desired state in the Git repository. If they match, the app is “Synced”; if not, it is “OutOfSync”.

### 5. How do you install ArgoCD on a Kubernetes cluster?
**Answer:**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## 🔶 Medium Level

### 1. Explain the difference between manual sync and automatic sync in ArgoCD.
**Answer:** 
- **Manual Sync:** Users must trigger synchronization explicitly.
- **Automatic Sync:** ArgoCD automatically applies changes when Git updates are detected.

### 2. What is an ArgoCD Project? Why would you use Projects in a multi-tenant environment?
**Answer:** Projects are used to group applications and enforce access and deployment restrictions. In a multi-tenant environment, Projects help isolate teams and restrict them to specific repos, clusters, or namespaces.

### 3. Describe the role of the ArgoCD Application Controller.
**Answer:** It monitors Applications, compares Git and cluster states, and performs syncing to align the cluster with the Git source.

### 4. What is the significance of Application health status in ArgoCD? List at least three health statuses.
**Answer:** Health status shows if an application is running correctly.
- `Healthy`
- `Degraded`
- `Progressing`

### 5. How can you customize deployment manifests using Helm charts in ArgoCD?
**Answer:** In the Application manifest, set `source.helm.parameters` or `values`. ArgoCD renders Helm charts before applying to the cluster.

### 6. Explain how RBAC (Role-Based Access Control) is implemented in ArgoCD.
**Answer:** ArgoCD uses a `policy.csv` file and role definitions to grant permissions to users and groups based on actions (get, create, sync, delete) and scopes (applications, projects, clusters).

---

## 🔴 Hard Level

### 1. Describe how ArgoCD can be integrated with SSO (Single Sign-On) providers.
**Answer:** ArgoCD integrates with SSO providers using Dex. Dex supports LDAP, GitHub, OIDC, etc. Configurations are set in `argocd-cm` and `argocd-rbac-cm`.

### 2. Explain how ArgoCD handles application lifecycle hooks. Provide an example use case for PreSync or PostSync hooks.
**Answer:** Hooks (PreSync, Sync, PostSync) are special annotations on Kubernetes Jobs or resources. Example: A `PreSync` job to back up a database before a migration.

### 3. Discuss the architecture of ArgoCD. How do its components interact to maintain application state?
**Answer:**
- `argocd-server`: UI & API
- `argocd-repo-server`: Clones repos, renders manifests
- `argocd-application-controller`: Compares & syncs state
- `argocd-dex-server`: Handles authentication

### 4. What are ApplicationSets in ArgoCD? How do they simplify multi-cluster or multi-environment deployments?
**Answer:** ApplicationSets allow templated creation of multiple Applications. Useful for deploying the same app to multiple clusters/environments using a single config.

### 5. How does ArgoCD manage rollback and history of application deployments?
**Answer:** ArgoCD stores revision history. Users can rollback to any previous revision via CLI or UI.

### 6. Explain the security best practices when using ArgoCD in a production environment.
**Answer:**
- Enable SSO and RBAC
- Limit network access to UI/API
- Use sealed secrets or external secret managers
- Monitor audit logs
- Enable TLS and use strong passwords

### 7. Discuss the differences and use cases for Kustomize vs Helm in ArgoCD.
**Answer:**
- **Helm:** Template engine, parameterized configs, charts
- **Kustomize:** Patch-based customization, declarative overlays
- Use Helm for complex templating; Kustomize for structured overlays.

---

## 📚 Case Study Answers

### 1. Multi-Cluster Deployment with ArgoCD ApplicationSets

**Design:** Use `ApplicationSet` with a `clusters` generator to target all registered clusters.

**Config overrides:** Use Helm value files or Kustomize overlays per cluster/environment.

**Benefits:** Consistency, automation.  
**Challenges:** Cluster authentication, complexity in overrides.

---

### 2. GitOps Workflow for a Microservices Architecture

**Structure:**
- Use one Application per service (frontend, backend, db)
- Group using Projects

**Sync & Dependencies:** Use sync waves or hooks for order.

**Monitoring:** Use ArgoCD UI/CLI for status and health.

---

### 3. Securing ArgoCD in a Production Environment

**RBAC Policy:** Limit roles to specific apps/namespaces (e.g., devs can’t delete production apps).

**Authentication:** Configure Dex with LDAP or OIDC providers.

**Network Policies:** Restrict access to ArgoCD server via Kubernetes NetworkPolicy or firewalls.

---

### 4. Disaster Recovery and Rollback with ArgoCD

**History:** ArgoCD records deployment revisions.

**Rollback Procedure:**
```bash
argocd app rollback myapp <revision>
```

**Safe Rollbacks:** Use `PreSync` and `PostSync` hooks to backup and validate before reverting.

---

### 5. Implementing CI/CD Pipeline with ArgoCD and GitHub Actions

**Integration:**
- GitHub Action pushes code, builds images
- Commits manifest updates to Git
- Optional: Trigger `argocd app sync` via CLI/API

**Sample Workflow:**
```yaml
- name: Sync ArgoCD App
  run: |
    argocd login $ARGOCD_SERVER --username admin --password $ARGOCD_PASSWORD
    argocd app sync myapp
```

**Pros:** Simpler, Git-centric, faster deploys  
**Cons:** Less control than Jenkins pipelines

---
