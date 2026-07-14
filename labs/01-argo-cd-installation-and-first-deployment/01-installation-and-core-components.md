
## 📄 `01-argo-cd-installation-and-first-deployment`

## Section 1: Installation, Architecture, First Login, and Understanding Core Components

### Learning Objectives

After completing this section, you will be able to:

* Install Argo CD on a Kubernetes cluster
* Understand every component installed by Argo CD
* Access the Argo CD UI
* Retrieve the default admin credentials
* Understand how Argo CD components interact internally
* Understand the role of each controller
* Visualize the GitOps workflow inside Argo CD
* Prepare your environment for deploying applications in later sections

## 1. Introduction

In the previous sections, we learned:

* What GitOps is
* GitOps principles
* Why GitOps is becoming the standard deployment methodology
* How Argo CD fits into the GitOps ecosystem
* Internal architecture of GitOps controllers

Now it is time to move from theory to practice.

> This section focuses on understanding Argo CD from a practical perspective.

Instead of discussing only concepts, we will:

* Install Argo CD
* Explore every component
* Understand how they communicate
* Access the UI
* Prepare for deploying real applications

> Think of this section as your first day working with Argo CD in a real DevOps environment.

## 2. Why Hands-On Learning Matters

Many engineers understand GitOps theoretically but struggle when they encounter:

* Installation issues
* UI access problems
* Authentication problems
* Sync failures
* Controller errors

Understanding architecture is important.

Using the product is even more important.

A DevOps engineer must be comfortable with:

* Installing Argo CD
* Troubleshooting Argo CD
* Managing applications
* Understanding logs
* Operating through both UI and CLI

This section lays that foundation.

## 3. Lab Environment Setup

For demonstration purposes, we will use:

| Component               | Value    |
| ----------------------- | -------- |
| Kubernetes Distribution | Minikube |
| Container Runtime       | Docker   |
| Git Repository          | GitHub   |
| GitOps Tool             | Argo CD  |
| Access Method           | UI + CLI |

Before we proceed with Argo CD installation, we must first set up a working local Kubernetes environment.

This requires two key tools:

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) (container runtime)
* [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download) (local Kubernetes cluster)

### 3.1 Installing Docker Desktop (Container Runtime)

We will use [Docker Desktop](https://www.docker.com/products/docker-desktop/) (container runtime) as the container runtime for running Kubernetes workloads locally.

Minikube requires a working container runtime to start and manage Kubernetes nodes.

#### Step 1: Download Docker Desktop

Go to the official Docker website:

* [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

Download the appropriate version for your operating system.

#### Step 2: Install Docker Desktop

Run the installer and follow the steps:

* Accept license agreement
* Enable WSL 2 integration (recommended for Windows)
* Enable virtualization if prompted
* Complete installation

#### Step 3: Start Docker Desktop

After installation:

* Launch [Docker Desktop](https://www.docker.com/products/docker-desktop/)
* Wait until status shows: `**Running**`
* Verify Docker is working

#### Step 4: Verify Installation

Open terminal (PowerShell or CMD):

```bash
docker version
```

Expected output:

```text
Client: Docker Engine
Server: Docker Engine
```

If both sections appear, Docker is installed successfully.

Also verify:

```bash
docker run hello-world
```

If you see a success message, Docker is correctly installed.

#### Why Docker Desktop is Required?

Docker provides:

* Container runtime
* Image management
* Local execution environment

> Without Docker:

* Minikube cannot start Kubernetes nodes (in most local setups)
* Container workloads will fail

#### Real-World Analogy

Think of Docker as the **engine of a car**.

Without an engine:

* The car (Kubernetes) cannot move

### 3.2 Installing Minikube (Local Kubernetes Cluster)

[Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download) allows you to run a local Kubernetes cluster on your workstation for development, testing, and learning purposes.

We will install Minikube and use it to create a local Kubernetes cluster.

#### Step 1: Install kubectl

Minikube works with Kubernetes CLI tool `kubectl`.

Install it using:

```bash
choco install kubernetes-cli
```

OR download from:

* [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

Verify:

```bash
kubectl version --client
```

#### Step 2: Install Minikube

> Option 1: Windows (Recommended)

Using Chocolatey:

```bash
choco install minikube
```

> Option 2: Direct Download

Download from:

* [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

Add binary to PATH.

#### Step 3: Start Minikube Cluster

Once installed, start the cluster:

```bash
minikube start --driver=docker
```

#### What is happening internally?

Minikube will:

* Create a lightweight VM or container
* Install Kubernetes inside it
* Configure `kubectl` automatically

### Step 4: Verify Minikube

```bash
minikube status
```

Expected output:

```text
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

#### Step 5: Verify Cluster Connectivity

```bash
kubectl get nodes
```

Expected output:

```text
NAME       STATUS   ROLES           AGE
minikube   Ready    control-plane   1m
```

> If the cluster is not available, Argo CD installation will fail.

### 3.3 How Docker + Minikube Work Together

When using Docker driver:

```text
Docker Desktop
        │
        ▼
Minikube Cluster
        │
        ▼
Kubernetes Nodes (containers inside Docker)
```

#### Simple Explanation

* Docker provides container runtime
* Minikube uses Docker to create Kubernetes nodes
* Kubernetes runs inside those containers


#### Real-World Analogy

Think of:

* Docker = foundation
* Minikube = house built on that foundation
* Kubernetes = rooms inside the house

### 3.4 Common Setup Issues

#### Issue 1: Docker not running

Error:

```text
Cannot connect to the Docker daemon
```

Fix:

* Start Docker Desktop
* Wait until status is "Running"

#### Issue 2: Virtualization disabled

Fix:

* Enable Virtualization in BIOS
* Enable WSL2 (Windows)

#### Issue 3: Minikube start fails

Fix:

```bash
minikube delete
minikube start --driver=docker
```

#### Issue 4: `kubectl` not found

Fix:

* Reinstall `kubectl`
* Add PATH variable

### 3.5 Final Verification Before Argo CD Installation

Run all checks:

```bash
docker version
kubectl version --client
minikube status
kubectl get nodes
```

If all commands return expected outputs:

> ✔ Your environment is READY for Argo CD installation

#### Summary of Lab Setup

Now your environment includes:

* ✓ [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
* ✓ [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download) installed and cluster started
* ✓ `kubectl` configured
* ✓ Kubernetes node visible
* ✓ Ready for [Argo CD](https://argoproj.github.io/cd/) installation

## 4. Installing Argo CD

[Argo CD](https://argoproj.github.io/cd/) supports multiple installation methods.

### Installation Method 1: Plain Kubernetes Manifests

This is the easiest and most commonly used approach for learning purposes.

#### Step 1: Create Namespace

```bash
kubectl create namespace argocd
```

Output:

```text
namespace/argocd created
```

Why Create a Dedicated Namespace?

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

Real-World Analogy

Imagine a large office building.

Instead of placing:

* HR
* Finance
* Security
* Engineering

inside one room,

you give each department its own floor.

Namespaces serve the same purpose.

#### Step 2: Install Argo CD

```bash
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

What Happens Internally?

This single command installs:

* Custom Resource Definitions (CRDs)
* Deployments
* Services
* ConfigMaps
* Secrets
* RBAC Objects
* Network Policies

Many beginners think Argo CD is a single deployment.

It is not.

> Argo CD is an ecosystem of multiple components working together.

## 5. Understanding the Installation Manifest

The installation file contains hundreds of Kubernetes resources.

When applied, Kubernetes creates:

```text
Namespace
│
├── Deployments
├── Services
├── Secrets
├── ConfigMaps
├── CRDs
├── Roles
├── RoleBindings
├── ServiceAccounts
└── Network Policies
```

### Why So Many Resources?

Argo CD performs many responsibilities:

* Git synchronization
* Kubernetes deployment
* Drift detection
* Authentication
* UI hosting
* RBAC
* Multi-cluster management

Each responsibility requires dedicated components.

## 6. Watching the Installation

After installation, monitor pods:

```bash
kubectl get pods -n argocd -w
```

### What Does `-w` Mean?

```bash
-w
```

stands for:

```text
Watch Mode
```

Instead of running repeatedly:

```bash
kubectl get pods
kubectl get pods
kubectl get pods
```

Kubernetes automatically refreshes the output.

Expected result:

```text
NAME                            READY   STATUS    AGE
argocd-server                   0/1     Pending   0s
argocd-repo-server              0/1     Pending   0s
argocd-application-controller   0/1     Pending   0s
argocd-dex-server               0/1     Pending   0s
argocd-redis                    0/1     Pending   0s
```

All should eventually become:

```text
Running
```

## 7. Argo CD Components Deep Dive

Now we arrive at one of the most important sections.

> Many engineers install Argo CD without understanding what each pod does.

When something breaks, they become lost.

Let's fix that.

### Component 1: Argo CD Server

Pod:

```text
argocd-server
```

#### Purpose

Acts as the API Server of Argo CD.

Every request goes through this component.

Examples:

* UI login
* CLI login
* Creating applications
* Deleting applications
* Syncing applications

#### Analogy

Think about Kubernetes.

When you execute:

```bash
kubectl apply
```

the request goes to:

```text
Kubernetes API Server
```

Similarly:

```bash
argocd app create
```

goes to:

```text
Argo CD Server
```

#### Responsibilities

* UI Hosting
* REST APIs
* Authentication
* Authorization
* CLI communication

### Component 2: Repo Server

Pod:

```text
argocd-repo-server
```

#### Purpose

Communicates with Git repositories.

Supported repositories:

* GitHub
* GitLab
* Bitbucket
* Azure Repos
* Any Git-compatible platform

#### Responsibilities

* Clone repositories
* Fetch manifests
* Process Helm charts
* Process Kustomize
* Process plugins

#### Analogy

Imagine a librarian.

Whenever someone requests a book:

The librarian fetches it from storage.

Repo Server behaves similarly.

Whenever Argo CD needs manifests:

Repo Server fetches them from Git.

### Component 3: Redis

Pod:

```text
argocd-redis
```

#### Purpose

> Caching.

Why caching?

Without Redis:

```text
Git → Download
Git → Download
Git → Download
Git → Download
```

for every request.

This becomes inefficient.

Redis stores frequently accessed data.

Benefits:

* Faster responses
* Reduced Git traffic
* Better performance

#### Real-World Analogy

Browser cache.

Instead of downloading a webpage every second:

The browser uses cached content.

Redis serves a similar purpose.

### Component 4: Application Controller

Pod:

```text
argocd-application-controller
```

#### Most Important Component

This is the heart of Argo CD.

Without it:

Argo CD becomes useless.

#### Responsibilities

* Compare Git state
* Compare Cluster state
* Detect drift
* Trigger synchronization
* Report health

#### GitOps Core Principle

Application Controller continuously checks:

```text
Desired State
vs
Actual State
```

#### Desired State

Git Repository

Example:

```yaml
replicas: 3
```

#### Actual State

Kubernetes Cluster

Example:

```yaml
replicas: 2
```

Application Controller detects:

```text
Out Of Sync
```

and initiates correction.

#### Analogy

Imagine a security guard.

The guard constantly checks:

```text
Checklist
vs
Reality
```

Whenever something differs:

  The guard fixes it.

Application Controller behaves exactly like this.

### Component 5: Dex Server

Pod:

```text
argocd-dex-server
```

#### Purpose

Authentication integration.

Supports:

* OIDC
* OAuth2
* SAML integrations

Examples:

* Google Login
* GitHub Login
* Azure AD
* Okta
* Keycloak

> Without Dex:

Only local accounts can authenticate.

> With Dex:

Enterprise SSO becomes possible.

#### Analogy

Dex is the receptionist verifying identities before granting building access.

### Component 6: Notifications Controller

Pod:

```text
argocd-notifications-controller
```

#### Purpose

Send notifications when events occur.

Examples:

```text
Application Failed
Application Synced
Health Degraded
Deployment Completed
```

Supported Targets

* Slack
* Microsoft Teams
* Email
* Webhooks

#### Example

A deployment fails.

Notification Controller sends:

```text
Slack Alert:
Guestbook deployment failed.
```

without requiring manual intervention.

### Component 7: ApplicationSet Controller

Pod:

```text
argocd-applicationset-controller
```

#### Purpose

Generate multiple applications automatically.

#### Problem

Suppose an organization manages:

```text
100 Clusters
```

Creating 100 applications manually is painful.

#### Solution

`ApplicationSet` Generates applications dynamically.

#### Analogy

Instead of manually creating 100 user accounts:

  You run a script.

ApplicationSet acts like that script.

## 8. Complete Architecture Diagram

```text
                  User
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
       UI                      CLI
        │                       │
        └───────────┬───────────┘
                    ▼
              Argo CD Server
                    │
     ┌──────────────┼──────────────┐
     │              │              │
     ▼              ▼              ▼
 Repo Server      Redis    Application Controller
     │                             │
     ▼                             ▼
 Git Repository            Kubernetes Cluster
```

## 9. Verifying Installation

Check deployments:

```bash
kubectl get deploy -n argocd
```

Check services:

```bash
kubectl get svc -n argocd
```

Check pods:

```bash
kubectl get pods -n argocd
```

All resources should show:

```text
Ready
Running
Available
```

## 10. Common Installation Problems

### Problem 1: Pods Stuck in Pending

**Possible causes:**

* Insufficient memory
* Insufficient CPU
* Node pressure

**Fix:**

```bash
# Check Minikube resources
minikube status

# If resources are low, increase them
minikube stop
minikube delete
minikube start --driver=docker --cpus=4 --memory=8192
minikube start
```

### Problem 2: Pods in CrashLoopBackOff

**Possible causes:**

* Corrupted installation
* Version mismatch
* Missing dependencies

**Fix:**

```bash
# Check pod logs
kubectl logs -n argocd <pod-name>

# Restart the pod
kubectl delete pod -n argocd <pod-name>

# If persistent, reinstall Argo CD
kubectl delete -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Problem 3: Namespace Missing

Verify:

```bash
kubectl get ns
```

**Fix:**

```bash
# Verify namespace exists
kubectl get ns argocd

# If missing, create it
kubectl create namespace argocd
```

### Problem 4: Minikube Not Running

Verify:

```bash
minikube status
```

**Fix:**

```bash
# Check Minikube status
minikube status

# If not running, start it
minikube start

# If already running but connection fails
minikube delete
minikube start --driver=docker
```

## Section Summary

In this section we:

* ✓ Installed Docker Desktop and Minikube
* ✓ Created a Kubernetes cluster
* ✓ Installed Argo CD using Kubernetes manifests
* ✓ Created a dedicated namespace
* ✓ Understood installation manifests
* ✓ Learned every core component
* ✓ Understood internal architecture
* ✓ Learned how GitOps reconciliation works
* ✓ Explored Application Controller
* ✓ Explored Repo Server
* ✓ Explored Dex
* ✓ Explored Redis
* ✓ Explored Notifications Controller
* ✓ Explored ApplicationSet Controller
* ✓ Verified successful installation
* ✓ Installed Argo CD CLI

## Key Takeaways

1. Argo CD is not a single pod — it is an ecosystem of multiple components
2. Application Controller is the core component responsible for GitOps reconciliation
3. Repo Server handles all Git repository interactions
4. Redis provides caching for improved performance
5. Dex enables enterprise authentication (OIDC/OAuth/SSO)
6. ApplicationSet automates large-scale deployments
7. Dedicated namespaces improve management and isolation
8. Understanding architecture simplifies troubleshooting
9. GitOps depends on continuous reconciliation between Git and cluster state
10. Installation success should always be verified before deployment

## Interview Questions and Answers

### 1. What is the role of Argo CD Server?

**Answer:** It acts as the API server for Argo CD and handles UI, CLI, authentication, and user requests.

### 2. Which component communicates with Git repositories?

**Answer:** Repo Server.

### 3. Which component performs reconciliation?

**Answer:** Application Controller.

### 4. Why does Argo CD use Redis?

**Answer:** For caching and performance optimization.

### 5. What is Dex?

**Answer:** An authentication component used for OIDC/OAuth/SSO integrations.

### 6. What is ApplicationSet?

**Answer:** A controller that automatically generates multiple Argo CD Applications for managing large-scale deployments.

### 7. Why should Argo CD be installed in a separate namespace?

**Answer:** Better isolation, management, monitoring, and troubleshooting.

### 8. What is the most critical component of Argo CD?

**Answer:** Application Controller because it maintains GitOps reconciliation between desired and actual state.

### 9. What happens if Git and Kubernetes states differ?

**Answer:** Argo CD marks the application OutOfSync and can automatically reconcile it to match the desired state.

### 10. What is the primary purpose of GitOps reconciliation?

**Answer:** Ensuring the actual cluster state continuously matches the desired state stored in Git.

## End of Section 1

**Next Section:** Accessing the Argo CD UI, Authentication, First Login, Deploying the Guestbook Application, and Understanding Argo CD Applications in Detail.
