
## 📄 `Multi-Cluster GitOps Deployment using Argo CD (Hub–Spoke Model on AWS EKS)`

## Section 7 — Deploying Applications Across Multiple Clusters

## 16. Argo CD Applications (Core Deployment Object)

### Learning Objectives

By the end of this section, you will understand:

* What an Argo CD Application is
* How Git repository maps to Kubernetes clusters
* How deployments are triggered automatically
* How multi-cluster deployments are defined

### 16.1 What is an Argo CD Application?

> An **Argo CD Application** is a Kubernetes Custom Resource Definition (CRD) that defines:

* Source (Git repository)
* Destination (Kubernetes cluster)
* Sync policy (manual/automatic)
* Path of manifests

### 16.2 Concept Mapping

| Concept     | Meaning                     |
| ----------- | --------------------------- |
| Git Repo    | Source of truth             |
| Path        | Folder containing manifests |
| Destination | Target cluster              |
| Sync Policy | Deployment behavior         |

### 16.3 Analogy

Argo CD Application is like:

> “A delivery instruction slip that tells Argo CD what to deploy, where to deploy, and how to keep it updated.”

### 16.4 Application YAML Structure

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/example/repo
    targetRevision: HEAD
    path: manifests/guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 16.5 Key Fields Explained

#### repoURL

Git repository containing Kubernetes manifests

#### path

Folder inside repo:

* guestbook manifests

#### destination

Target cluster + namespace

#### syncPolicy

Controls automation:

* `automated` → auto deploy
* `selfHeal` → fixes drift
* `prune` → deletes removed resources

## 17. Multi-Cluster Deployment Execution

### 17.1 Lab Flow

We now deploy:

* Guestbook application
* Across spoke clusters
* Using Argo CD hub

### 17.2 Deployment Steps

1. Open Argo CD UI
2. Click “Create Application”
3. Fill required fields:

   * Name
   * Repo URL
   * Path
   * Cluster destination
4. Click Create

### 17.3 Two Applications Created

* guestbook-1 → Spoke Cluster 1
* guestbook-2 → Spoke Cluster 2

### 17.4 Why Two Applications?

Because:

* Each application targets one cluster
* Each cluster has independent deployment object

## 18. GitOps Sync & Self-Healing

### 18.1 Auto Sync Behavior

When enabled:

* Argo CD automatically deploys changes from Git
* No manual intervention required

### 18.2 Self-Healing Concept

If someone modifies cluster manually:

```bash
kubectl edit configmap
```

Argo CD will:

* Detect drift
* Compare with Git
* Revert changes automatically

### 18.3 Drift Detection Flow

```id="drift1"
Git State → Compare → Cluster State → Mismatch → Auto Fix
```

### 18.4 Analogy

Self-healing is like:

> “A security guard restoring original order whenever someone changes rules illegally.”

## 19. Practical Demo: Guestbook Deployment

### 19.1 What Gets Deployed

* Deployment YAML
* Service YAML
* ConfigMap

### 19.2 Kubernetes Objects

#### Deployment:

Runs application pods

#### Service:

Exposes application

#### ConfigMap:

Stores configuration values

### 19.3 Verification Commands

```bash
kubectl get all
kubectl get configmap
kubectl get svc
```

### 19.4 Result

Application is deployed across:

* Spoke cluster 1
* Spoke cluster 2

## 20. Manual Change vs GitOps Correction

### 20.1 Scenario

Developer changes live config:

```bash
kubectl edit configmap guestbook-config
```

Change:

```yaml
UI_MODE: "test"
```

Git still has:

```yaml
UI_MODE: "production"
```

### ⚠️ 20.2 What Happens

Argo CD:

* Detects mismatch
* Marks application as “OutOfSync”

### 20.3 Auto Fix

If auto-heal is enabled:

* Reverts change automatically
* Restores Git state

### 20.4 Key Insight

> Git is always the source of truth, not the cluster.

## 21. Production Best Practices

### 21.1 Recommended Setup

* Always use HTTPS for Argo CD
* Enable RBAC
* Use HA mode for hub cluster
* Use separate repos per environment

### 21.2 Security Best Practices

* Never expose Argo CD publicly without security
* Use IAM roles for cluster access
* Restrict CLI access
* Use SSO (Dex/OIDC)

### 21.3 Scaling Best Practices

* Use ApplicationSets for large clusters
* Avoid single point of failure
* Monitor Redis + controller load

## 22. ApplicationSets (Next Evolution)

### 22.1 Problem

If you have:

* 100 clusters
* 100 applications

Manual creation becomes impossible

### 22.2 Solution

> ApplicationSet automatically generates applications dynamically.

### 22.3 Concept

Instead of:

* Creating 100 applications manually

You define:

* 1 ApplicationSet → generates all

## 23. Interview Revision Notes

### Q1: What is Argo CD Application?

**A:** A CRD that defines Git source, destination cluster, and sync policy.

### Q2: What is self-healing in Argo CD?

**A:** Automatic correction of drift between Git and cluster state.

### Q3: What is GitOps?

**A:** A methodology where Git is the single source of truth.

### Q4: What is Hub-Spoke model?

**A:** Central Argo CD managing multiple Kubernetes clusters.

### Q5: Why use multi-cluster architecture?

**A:** For isolation, scalability, and environment separation.

## FINAL SUMMARY

We covered:

### Concepts

* CI/CD limitations
* GitOps principles
* Argo CD architecture

### Architectures

* Hub-Spoke model
* Standalone model

### AWS EKS Lab

* Cluster creation
* CloudFormation debugging
* Context switching

### Argo CD Setup

* Installation
* UI access
* CLI login
* Cluster registration

### Deployment

* Applications
* Guestbook app
* Multi-cluster sync

### Advanced Features

* Self-healing
* Drift detection
* Auto sync
* ApplicationSets

## Congratulations! 🎉 

You have successfully completed **Lab 02: Multi-Cluster GitOps Deployment using Argo CD (Hub–Spoke Model on AWS EKS)**.

You now understand how to design, build, configure, and operate a production-style multi-cluster GitOps environment using Argo CD and Amazon EKS.
