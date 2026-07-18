
## 📄 GitOps Multi-Cluster Deployment with Argo CD (Hub–Spoke Model on AWS EKS)

## Section 6 — Connecting Spoke Clusters to the Hub

## 12. Argo CD Login & Authentication (UI + CLI)

### Learning Objectives

By the end of this section, you will understand:

* How to access Argo CD UI
* How initial admin password is generated
* How to decode Kubernetes secrets (base64)
* How CLI login works
* Why authentication is required before cluster operations

### 12.1 Argo CD Login Overview

After installation, Argo CD provides:

* Web UI login
* CLI login (`argocd login`)

Both use:

* `admin` username
* Password stored in Kubernetes secret

### 12.2 Finding Initial Admin Password

Argo CD automatically creates an admin account.

**Username:**

```text
admin
```

**Password:**

Stored inside Kubernetes Secret.

**View secrets:**

```bash
kubectl get secrets -n argocd
```

output:

```text
NAME                          TYPE     DATA   AGE
argocd-initial-admin-secret   Opaque   1      5m
argocd-secret                 Opaque   1      5m
argocd-repo-server            Opaque   1      5m
```

You should find:

```text
argocd-initial-admin-secret
```

### 12.3 Extract Secret

**View secret:**

```bash
kubectl edit secret argocd-initial-admin-secret -n argocd
```

Or:

```bash
kubectl get secret argocd-initial-admin-secret \
-n argocd -o yaml
```

Example:

```yaml
data:
  password: YWJjMTIzNDU=
```

The value is Base64 encoded.

### 12.4 Why Base64 Encoding?

Kubernetes stores secrets in:

> base64 encoded format (not encrypted)

So we must decode it.

### 12.5 Decode Password

```bash
echo <encoded-password> | base64 --decode
```

Example:

```bash
echo YWJjMTIzNDU= | base64 --decode
```

Example Output:

```text
abc12345
```

#### Result:

You get the real admin password.

#### Analogy

Base64 is like:

> “A simple lock (not real encryption) used to format data safely, not secure it.”

### 12.6 Login to Argo CD UI

* Open browser
* Enter:

  * Username: `admin`
  * Password: decoded value

## 13. Argo CD CLI Login

### 13.1 Install CLI

#### macOS:

```bash
brew install argocd
```

#### Linux:

Download binary from official release.

### 13.2 CLI Login Command

```bash
argocd login <ARGOCD_SERVER_IP>:<PORT>
```

Example:

```bash
argocd login 3.92.12.45:30812
```

### 13.3 Authentication Required

You must provide:

* Username: admin
* Password: decoded secret

### 13.4 Why CLI Login Matters

CLI is used for:

* Adding clusters
* Automation scripts
* CI/CD integration
* Bulk operations

## 14. Adding Kubernetes Clusters to Argo CD

### Learning Objectives

* Understand cluster registration
* Use `argocd cluster add`
* Connect hub to spoke clusters
* Understand kubeconfig context mapping

### 14.1 Why Add Clusters?

By default:

* Argo CD can only deploy to its own cluster

To enable multi-cluster:

* We register external clusters

### 14.2 Check Available Contexts

```bash
kubectl config get-contexts
```

You will see:

* hub cluster
* spoke cluster 1
* spoke cluster 2

### 14.3 Add Cluster Command

Run:

```bash
argocd cluster add <context-name>
```

#### Example:

```bash
argocd cluster add eks-spoke-1
```

#### What Happens Internally?

Argo CD will:

* Create ServiceAccount in target cluster
* Create RBAC permissions
* Store cluster credentials
* Register cluster in Argo CD

### 14.4 Architecture Flow

```id="flow1"
Argo CD (Hub)
     │
     ├── connects using kubeconfig
     │
     ▼
Spoke Cluster 1
Spoke Cluster 2
```

### ⚠️ 14.5 Common Error

#### Error:

* CLI login not done
* Permission denied

#### Fix:

```bash
argocd login <server>
```

Then retry cluster add.

#### Analogy

Cluster registration is like:

> “Giving a central office access cards to branch offices.”

## 15. Hub Cluster Control Plane Behavior

### 15.1 What Happens After Cluster Add?

Argo CD can now:

* Deploy applications to multiple clusters
* Monitor cluster health
* Sync state across environments

### 15.2 Final Multi-Cluster Setup

```id="final1"
           Git Repo
              │
              ▼
        Argo CD (Hub)
          ├──────────────┐
          ▼              ▼
   Spoke Cluster 1   Spoke Cluster 2
```

### 15.3 Key Insight

* Hub cluster runs Argo CD
* Spoke clusters are targets
* One Argo CD can manage many clusters

### Interview Questions

#### Q1: Where is Argo CD password stored?

**A:** In Kubernetes secret `argocd-initial-admin-secret`.

#### Q2: Why do we decode base64 secrets?

**A:** Because Kubernetes stores secrets in base64 encoding.

#### Q3: What does `argocd cluster add` do?

**A:** It registers an external Kubernetes cluster with Argo CD.

#### Q4: Why is CLI required?

**A:** For automation and cluster management operations not available in UI.

### Summary

* Argo CD login uses admin + secret password
* Password is base64 encoded in Kubernetes secret
* CLI login enables advanced operations
* `argocd cluster add` connects spoke clusters
* Hub cluster becomes centralized control plane

## ➤ Next: Section 7 — FINAL

We will cover the most important execution phase:

* Understanding Argo CD Applications
* Git Repository Structure
* Sync Policy (Manual vs Automatic)
* Self-Healing Behavior
* Drift Detection Demo
* Multi-Cluster Deployment Workflow (Guestbook Deployment)
* Final Interview Revision Notes
* Production Best Practices
