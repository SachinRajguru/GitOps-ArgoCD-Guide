
## 📄 GitOps Multi-Cluster Deployment with Argo CD (Hub–Spoke Model on AWS EKS)

## Section 5 — Installing Argo CD on the Hub Cluster

## 9. Kubernetes Context Switching (Multi-Cluster Control)

### Learning Objectives

By the end of this section, you will understand:

* How kubeconfig manages multiple clusters
* What Kubernetes context is
* How to switch between Hub and Spoke clusters
* Why correct context selection is critical in multi-cluster setups

### 9.1 What is a Kubernetes Context?

> A **context** in Kubernetes is a configuration entry that defines:

* Which cluster you are connected to
* Which user credentials are used
* Which namespace is default

### 9.2 Why Context Matters

In multi-cluster environments:

* You may have 10–100 clusters
* One wrong command can affect production instead of dev

➤ So context switching is critical.

### 9.3 Viewing Available Contexts

Run:

```bash
kubectl config get-contexts
```

#### Output includes:

* hub-cluster
* spoke-cluster-1
* spoke-cluster-2

### 9.4 Selecting Region-Specific Clusters

Example:

* Filtering by region (us-west-1)

```bash
kubectl config get-contexts | grep us-west-1
```

### 9.5 Switching Context

```bash
kubectl config use-context hub-cluster
```

#### Result:

Now all kubectl commands apply to:

> HUB cluster (where Argo CD is installed)

#### Analogy

Kubernetes context is like:

> “Changing remote control from TV in Room A to Room B”

Same commands, different target system.

### ⚠️ 9.6 Common Mistake

Developers often forget:

* Which cluster is active
* They deploy into wrong environment

➤ This is a major production risk.

## 10. Installing Argo CD on Hub Cluster

### Learning Objectives

* Install Argo CD using Kubernetes manifests
* Understand Argo CD namespace structure
* Verify deployment components
* Understand why installation is cluster-specific

### 10.1 Installation Method

We use official Argo CD YAML manifests.

### 10.2 Create Namespace

```yaml
kubectl create namespace argocd
```

Output:

```
namespace/argocd created
```

#### Why Create a Dedicated Namespace?

Best practice in Kubernetes is to isolate controllers.

Examples:

| Controller | Namespace    |
| ---------- | ------------ |
| Argo CD    | argocd       |
| Istio      | istio-system |
| Prometheus | monitoring   |
| Jenkins    | jenkins      |

Benefits:

* Better organization
* Easier troubleshooting
* Easier monitoring
* Reduced risk of accidental deletion

### 10.3 Install Argo CD Core Components

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### What gets installed?

Inside `argocd` namespace:

* API Server
* Application Controller
* Repo Server
* Dex (Authentication)
* Redis

### 10.4 Argo CD Pod Architecture

```id="pods1"
argocd namespace:
 ├── argocd-server
 ├── argocd-repo-server
 ├── argocd-application-controller
 ├── argocd-dex-server
 └── argocd-redis
```

### 10.5 Checking Installation

```bash id="check1"
kubectl get pods -n argocd
```

### 10.6 Key Insight

> Argo CD is installed ONLY on HUB cluster

Because:

* It acts as centralized control plane
* It manages spoke clusters

## 11. Exposing Argo CD UI (NodePort Access)


### Learning Objectives

* Understand service exposure methods
* Convert ClusterIP → NodePort
* Access Argo CD UI via EC2 public IP

### 11.1 Service Types in Kubernetes

* ClusterIP (default, internal only)
* NodePort (external access via node IP)
* LoadBalancer (cloud-managed)
* Ingress (advanced routing)

### 11.2 Why NodePort in This Project?

Reason:

* Simplicity
* EC2-based cluster
* Faster access

> “We are not using Ingress today”

### 11.3 Change Service Type

One common way to expose Argo CD is to convert the `argocd-server` service from a **ClusterIP** service into a **NodePort** service.

Locate the service:

```bash
kubectl get svc -n argocd
```

Find:

```text
argocd-server
```

Edit it:

```bash
kubectl edit svc argocd-server -n argocd
```

Locate:

```yaml
type: ClusterIP
```

Change to:

```yaml
type: NodePort
```

Save and exit.

#### What Happens Internally?

Before:

```text
User
  │
  X
ClusterIP
```

After:

```text
User
  │
NodePort
  │
Argo CD Server
```

Now external traffic can reach Argo CD.

#### Why Does Argo CD Server Need Exposure?

Remember from Section 1:

Argo CD Server acts as:

* UI endpoint
* CLI endpoint
* API endpoint

Without access to this service:

* UI won't work
* CLI won't work

### 11.4 Access Flow

```id="access1"
Browser → EC2 Node IP → NodePort → Argo CD UI
```

### ⚠️ 11.5 Security Group Requirement

You must open:

* NodePort range (example: 30000–32767)

Otherwise UI will not load.

#### Analogy

NodePort is like:

> “Opening a specific door on a building so outsiders can enter directly.”

### 11.6 Final Result

Argo CD UI becomes accessible via:

```
http://<EC2_PUBLIC_IP>:<NODE_PORT>
```

### Interview Questions

#### Q1: What is Kubernetes context?

**A:** A configuration that defines cluster, user, and namespace for kubectl operations.

#### Q2: Why install Argo CD only on hub cluster?

**A:** Because hub cluster acts as centralized control plane for multi-cluster deployment.

#### Q3: What is NodePort?

**A:** A Kubernetes service type that exposes applications on a static port across all nodes.

#### Q4: Why is context switching important?

**A:** To ensure kubectl commands target the correct cluster.

### Summary

* Kubernetes context controls cluster targeting
* kubeconfig stores multiple cluster configs
* Argo CD is installed only on hub cluster
* UI is exposed using NodePort in this project
* EC2 security groups must allow traffic

## ➤ Next: Section 6

We will cover the most critical operational steps:

* Argo CD Login (CLI and Web UI)
* Extracting the Initial Admin Password
* Adding Spoke Clusters to the Hub Cluster
* Understanding `argocd cluster add`
* Cluster Registration Workflow
