
# GitOps Architecture

## Understanding Argo CD, Flux CD, and the Internal Architecture of GitOps Controllers

## Table of Contents

1. [Introduction to the GitOps Architecture](#1-introduction-to-the-gitops-architecture)
2. [Prerequisites for Understanding GitOps Architecture](#2-prerequisites-for-understanding-gitops-architecture)
3. [Popular GitOps Tools](#3-popular-gitops-tools)
4. [History of Argo Project (Quick Overview)](#4-history-of-argo-project-quick-overview)
5. [Understanding GitOps Tool Philosophy (High-Level View)](#5-understanding-gitops-tool-philosophy-high-level-view)
6. [Reconciliation Logic in Kubernetes Controllers](#6-reconciliation-logic-in-kubernetes-controllers)
7. [Architecture of GitOps Tools (Core Design)](#7-architecture-of-gitops-tools-core-design)
8. [Installation Approaches](#8-installation-approaches)
9. [Real-World GitOps Scenarios](#9-real-world-gitops-scenarios)
10. [Argo CD vs Flux CD](#10-argo-cd-vs-flux-cd)
11. [Final Summary](#11-final-summary)
12. [Key Takeaways](#12-key-takeaways)
13. [Interview Questions](#13-interview-questions)
14. [What Comes Next](#14-what-comes-next)

## 1. Introduction to the GitOps Architecture

Welcome to the next stage of learning GitOps.

If the first step was understanding **what GitOps is**, the second step is understanding **how GitOps actually works internally**.

This chapter is focused on architecture.

By the end of this learning journey, you will be able to:

* Explain GitOps architecture confidently
* Understand how GitOps controllers are built
* Switch between tools like Argo CD and Flux CD
* Explain GitOps to others
* Answer architecture-focused interview questions

This is where GitOps stops being just a buzzword and starts becoming a real engineering concept.

### Why Architecture Matters

Most learners stop here:

> "GitOps means deploying Kubernetes manifests from Git."

That is only the surface.

The real understanding begins when you ask:

* How does a GitOps tool know something changed?
* How does it compare Git and Kubernetes?
* What components are involved?
* How does synchronization happen?
* How does self-healing work?

To answer these, we need architecture.

### Real-World Analogy

Think of GitOps like **autopilot in an aircraft**.

The pilot sets the desired destination.

| Component         | Analogy                   |
|-------------------|---------------------------|
| Git               | Desired destination       |
| Kubernetes        | Current aircraft position |
| GitOps Controller | Autopilot system          |

The autopilot continuously checks:

> "Am I where I am supposed to be?"

If not, it corrects course automatically.

That is reconciliation.

### Mini Summary

GitOps architecture helps us understand:

* Internal mechanics
* Automation flow
* Self-healing systems
* Kubernetes controller behavior

## 2. Prerequisites for Understanding GitOps Architecture

Before architecture, foundational principles must be understood.

* GitOps Fundamentals established the "why."
* GitOps Architecture explains the "how."

### What Was Covered Previously

The foundational concepts include:

### 1. What is GitOps?

GitOps is an operational framework where:

* Git stores desired infrastructure/application state
* Automated controllers reconcile actual state with desired state

### 2. Why GitOps?

GitOps provides:

* Declarative deployments
* Version control
* Auditability
* Automated rollback
* Self-healing

### 3. GitOps Principles

A tool qualifies as GitOps if it follows:

**Declarative configuration**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

**Version-controlled source**

> Stored in Git repository.

**Automated synchronization**

> Controllers monitor changes.

**Continuous reconciliation**

> System corrects drift automatically.

### 4. Understanding Terraform vs GitOps

> ⚠️ **Important nuance:** Argo CD is designed as a GitOps tool, while Terraform is primarily an infrastructure provisioning tool.

| Aspect              | Argo CD (GitOps)           | Terraform                   |
|---------------------|----------------------------|-----------------------------|
| **Primary Focus**   | Continuous synchronization | Infrastructure provisioning |
| **Reconciliation**  | Continuous (runs forever)  | On-demand (runs and exits)  |
| **Drift Detection** | Automatic and continuous   | Requires explicit plan/run  |
| **Target**          | Kubernetes resources       | Any infrastructure          |
| **Loop Behavior**   | Watch loop                 | Apply and exit              |

```bash
# Argo CD - continuously watches and reconciles
argocd app sync my-app
# Continuously monitors cluster state

# Terraform - runs on demand
terraform apply
# Executes once and exits
```

**Terraform CAN participate in GitOps workflows** when:

* Uses remote state (Terraform Cloud/Enterprise)
* Combined with tools like Spacelift, Atlantis, or Env0
* Uses "plan/apply" separation with code review
* Implements continuous detection via external automation

However, Terraform is still primarily infra-provisioning focused, not continuous reconciliation by design.

### Mini Summary

Architecture only makes sense after understanding:

* GitOps principles
* Desired state management
* Reconciliation loops

## 3. Popular GitOps Tools

Before diving into architecture, it is important to understand the ecosystem of GitOps tools.

Several tools exist in the GitOps ecosystem.

Understanding them helps compare architectures.

### Concept Explanation

Today's DevOps ecosystem has several GitOps-enabled tools:

### 1. Argo CD

[Argo CD](https://argo-cd.readthedocs.io/en/stable/)

> The most widely adopted GitOps tool.

**Why popular:**

* Rich UI for visualization
* Kubernetes-native design
* Strong and active community support
* CNCF Graduated Project
* Easy onboarding and operational experience

### 2. Flux CD (v2)

> ⚠️ **Important:** Flux v1 is archived. Current version is **Flux v2**.

[Flux](https://fluxcd.io/flux/)

CNCF Graduated GitOps tool with modular architecture.

**Known for:**

* Highly modular design (GitOps Toolkit)
* CRD-based approach
* Strong automation capabilities
* Active development (Flux v2)

**Flux v2 Architecture:**

```
┌────────────────────────────┐
│    GitOps Toolkit          │
├────────────────────────────┤
│ Source Controller          │ → Watches Git
│ Kustomize Controller       │ → Builds manifests
│ Helm Controller            │ → Helm releases
│ Notification Controller    │ → Alerts
└────────────────────────────┘
```

### 3. Jenkins X (Archived)

[Jenkins X](https://github.com/jenkins-x/jx-api)

> ⚠️ **Important Note:** Jenkins X was **archived in 2022** and is no longer actively maintained.

**Historical context:**

* Originally designed for Kubernetes-based CI/CD with GitOps
* Combined CI/CD pipelines with GitOps principles
* Now in maintenance mode (read-only)

> **For new projects, use Argo CD or Flux CD instead.**

### 4. Spinnaker

[Spinnaker](https://spinnaker.io/)

> Primarily a deployment-oriented platform.

**Key insight:**

> Spinnaker ≠ pure GitOps tool

It is primarily a continuous delivery platform that **can support GitOps workflows** via integration, but was not designed solely around GitOps principles.

### Comparison Table

| Tool       | Status          | GitOps Purity | Focus                          |
|------------|-----------------|---------------|--------------------------------|
| Argo CD    | Active          | High          | GitOps-first (continuous sync) |
| Flux CD v2 | Active          | High          | GitOps-first (modular)         |
| Jenkins X  | Archived (2022) | Medium        | CI/CD + GitOps (legacy)        |
| Spinnaker  | Active          | Low           | Deployment-first               |

### Analogy

Think of these like transportation options:

| Tool       | Analogy             | Description                      |
|------------|---------------------|----------------------------------|
| Argo CD    | Luxury sedan        | Feature-rich, user-friendly      |
| Flux CD v2 | Modular sports bike | Lightweight, highly customizable |
| Jenkins X  | Retired utility van | No longer in production          |
| Spinnaker  | Cargo truck         | Heavy-duty deployment platform   |

Different designs, different priorities and operational trade-offs.

### Mini Summary

The GitOps ecosystem includes multiple tools, but **Argo CD and Flux v2** dominate because they are actively maintained and purpose-built for GitOps.

## 4. History of Argo Project (Quick Overview)

Understanding history helps explain why Argo CD became so successful.

### Origin

Argo CD was originally created by engineers at: **Applatix**

It was later open-sourced and gained rapid adoption in the cloud-native community.

### Acquisition

Applatix was acquired by: **Intuit**

The project continued to evolve under strong enterprise and open-source contributions

### The Argo Ecosystem

The broader Argo project includes multiple projects:

| Project            | Purpose                                   |
|--------------------|-------------------------------------------|
| Argo CD            | GitOps continuous delivery                |
| Argo Rollouts      | Progressive delivery / canary deployments |
| Argo Workflows     | Workflow orchestration                    |
| Argo Events        | Event-driven automation                   |
| Argo Notifications | Notification management                   |
| Argo Image Updater | Automatic container image updates         |

### Contributors

Major contributors include:

* BlackRock
* Red Hat
* Codefresh
* Intuit
* Akuity
* Many others from the CNCF community

### CNCF Graduation

Argo CD is a:

> Cloud Native Computing Foundation [(CNCF)](https://www.cncf.io/) Graduated Project

This indicates:

* Maturity
* Security
* Stability
* Strong governance

### Mini Summary

Argo CD's success is rooted in:

* Strong open-source backing
* Active contributors
* CNCF ecosystem integration

## 5. Understanding GitOps Tool Philosophy (High-Level View)

Now we simplify the entire concept into one mental model.

### Concept Explanation

A GitOps tool's job is simple in principle:

> Maintain continuous synchronization between Git and Kubernetes.

Where:

* Git = Desired State
* Kubernetes = Live State

### The Core Model

```
+-------------------+
|      Git Repo     |
+-------------------+
         |
         v
+-------------------+
|   GitOps Tool     |
+-------------------+
         |
         v
+-------------------+
| Kubernetes Cluster|
+-------------------+
```

### What Happens?

The tool continuously ensures:

> Kubernetes state == Git state

### How It Works

1. Git contains deployment manifests
2. GitOps tool reads these manifests
3. It applies them to Kubernetes
4. It continuously compares both states
5. If drift is found → it corrects it

### Example

Git contains:

```yaml
replicas: 3
```

Cluster accidentally changes to:

```yaml
replicas: 5
```

GitOps detects drift.

It restores:

```yaml
replicas: 3
```

### Analogy

Think of Git as a "rule book" and Kubernetes as "students in a classroom."

Even if students try to change rules on their own:

* Teacher (GitOps controller) enforces the rulebook again

### Mini Summary

At a high level, GitOps continuously synchronizes Git and Kubernetes and automatically corrects any drift between them.

## 6. Reconciliation Logic in Kubernetes Controllers

> GitOps tools are fundamentally Kubernetes controllers.

Their most important capability is reconciliation.

### What is Reconciliation?

Reconciliation means:

> Continuously comparing actual state with desired state and correcting deviations.

### Controller Loop

```
Observe → Compare → Correct → Repeat
```

### Pseudocode

```go
// Simplified GitOps reconciliation loop
for {
   // 1. Observe: Get desired state
   desired := getGitState()
   
   // 2. Observe: Get actual state
   actual := getClusterState()
   
   // 3. Compare: Detect drift
   if desired != actual {
      // 4. Correct: Reconcile
      reconcile()
   }
   
   // 5. Repeat: Wait and check again
   sleep(reconciliationInterval)
}
```

### Why It Matters

| Traditional Scripts          | GitOps Controllers         |
|------------------------------|----------------------------|
| Run once                     | Run continuously           |
| Exit after completion        | Continuous monitoring      |
| No drift detection           | Automatic drift correction |
| Manual intervention required | Self-healing               |

### Key Components of Reconciliation

1. **Observation** - Gathering both desired and actual state
2. **Comparison** - Calculating differences
3. **Decision** - Determining what needs to change
4. **Action** - Applying corrections
5. **Observation** - Verifying the apply worked

### Mini Summary

Reconciliation is the heart of GitOps.

Without reconciliation, there is no self-healing, drift correction, or continuous state enforcement.

## 7. Architecture of GitOps Tools (Core Design)

### Introduction

Now we move into the most important part: `how GitOps tools are built internally.`

We will use `Argo CD architecture` as the reference model.

### Concept Explanation

A GitOps system is not a single service. It is a collection of microservices.

At a high level, it has three responsibilities:

1. Read Git state
2. Read Kubernetes state
3. Compare and sync both states

### Full Architecture

```
             ┌──────────────┐
             │      Git     │
             └───────┬──────┘
                     ▼
           ┌──────────────────┐
           │    Repo Server   │
           │  (Git Interface) │
           └─────────┬────────┘
                     ▼
      ┌─────────────────────────────┐
      │   Application Controller    │
      │   (Reconciliation Engine)   │
      └───────┬─────────────┬───────┘
              ▼             ▼
         ┌──────────┐  ┌──────────┐
         │ K8s API  │  │  Redis   │
         │ Server   │  │  Cache   │
         └────┬─────┘  └────┬─────┘
              │             │
              │       ┌─────┴──────┐
              │       ▼            ▼
              │ ┌──────────┐ ┌──────────┐
              │ │AppSet    │ │Notif     │
              │ │Controller│ │Controller│
              │ └──────────┘ └──────────┘
              │
              ▲
              │
       ┌──────┴──────┐
       │  API Server │
       │  (UI + CLI) │
       └──────┬──────┘
              ▼
             User

        ┌─────────────┐
        │     Dex     │
        │ (SSO Layer) │
        └─────────────┘
```

This architecture contains multiple microservices.

Each has a specific responsibility.

### Mini Summary

Argo CD is not a single binary doing everything.

It is a distributed microservice-based system.

## Breaking Down Argo CD Architecture

### 1. Repo Server

> The Repo Server communicates with Git.

**Responsibility:**

It:

* Connects to Git repositories
* Pulls manifests
* Parses YAML/Helm/Templating
* Generates Kubernetes manifests

**Flow:**

```
Git Repository
      │
      │ (HTTPS/SSH)
      ▼
┌─────────────┐
│ Repo Server │ ── Parses YAML
└─────────────┘
      │
      │ (Manifests)
      ▼
Application Controller
```

**Example:**

Repo contains:

```yaml
deployment.yaml
service.yaml
configmap.yaml
```

Repo Server fetches and processes them.

**Analogy:**

Repo Server is the **librarian**.

It retrieves the correct book (manifest) when requested.

**Mini Summary:**

Repo Server is Git-facing.

It knows desired state.

### 2. Application Controller

> This is the brain of Argo CD.

**Responsibilities:**

It:

* Reads cluster state
* Compares desired vs actual
* Detects drift
* Initiates synchronization

**Reconciliation Flow:**

```
Desired State (from Repo Server)
          │
          ▼
┌─────────────────────┐
│ Compare with        │ ──── Detect Drift
│ Live Cluster State  │
└─────────────────────┘
          │
          ▼ (if drift detected)
┌─────────────────────┐
│ Apply Corrections   │
└─────────────────────┘
```

**Example Workflow:**

1. Repo Server fetches Git manifests
2. Controller checks Kubernetes
3. Detects mismatch
4. Applies correction

**Analogy:**

The Application Controller is the **quality inspector**.

It ensures what should be in the cluster actually is.

**Mini Summary:**

The controller performs reconciliation.

### 3. ApplicationSet Controller (v2.4+)

> Enables set-based deployments.

**Responsibilities:**

* Generate multiple Applications from templates
* Handle cluster fleet management
* Support matrix/applying generators

**Example:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-appset
spec:
  generators:
  - matrix:
      generators:
      - git:
          repoURL: https://github.com/org/repo
          revision: HEAD
          directories:
          - path: apps/*
      - clusters:
          values: clusterName
```

**Use Case:**

Deploying the same application to 50 clusters becomes one ApplicationSet instead of 50 individual Applications.

**Mini Summary:**

ApplicationSet controller handles fleet-scale deployments.

### 4. API Server

> User interface layer (UI + CLI)

Users need a way to interact with Argo CD.

That interface is the API Server.

**It Provides:**

* Web UI
* CLI access
* Authentication
* Authorization
* REST/gRPC API

**Example CLI:**

```bash
# Login to Argo CD
argocd login argocd.example.com

# Sync an application
argocd app sync my-app

# View application status
argocd app get my-app
```

**Mini Summary:**

API Server is the human interaction layer.

### 5. Notifications Controller

> Handles alerts and notifications.

**Responsibilities:**

* Send notifications on sync status changes
* Notify on deployment failures
* Integration with Slack, Email, Teams, etc.

**Supported Destinations:**

* Slack
* Email
* Microsoft Teams
* Discord
* Webhooks

**Mini Summary:**

Notifications controller keeps teams informed.

### 6. Dex and Authentication

> Enterprise environments require SSO.

**What Dex Does:**

[Dex](https://dexidp.io/) enables:

* OIDC
* OAuth
* SSO integration

**Supported Identity Providers:**

* Google
* LDAP
* GitHub
* SAML providers
* Azure AD

**Flow:**

```
User → API Server → Dex → Identity Provider
                ←─── Token ────
```

**Analogy:**

Dex is a **translator** between Argo CD and identity systems.

**Mini Summary:**

Dex provides enterprise-grade authentication integration.

### 7. Redis and State Caching

> State must persist.

**Why Redis?**

[Redis](https://redis.io/) stores:

* Cache
* Session data
* State snapshots
* History

**Why Important?**

* Fast state recovery after restart
* Reduces API server load
* Stores application history

> ⚠️ **Note:** Redis is optional for small deployments but recommended for production.

**Mini Summary:**

Redis improves resilience and recovery.

## 8. Installation Approaches

Argo CD can be installed in multiple ways.

### Method 1: YAML Manifests

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Method 2: Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd
```

### Method 3: Operator

Operator-managed lifecycle automation.

```bash
kubectl apply -f https://operatorhub.io/install/argocd-operator.yaml
```

### Method 4: Cluster Add-on

Managed Kubernetes services often provide Argo CD as an add-on.

### Real-World Note

Most production systems prefer:

> Helm or operator-based installation

This provides:

* Easier upgrades
* Declarative configuration
* Lifecycle management

### Mini Summary

Different installation methods suit different operational needs.

## 9. Advanced Real-World GitOps Scenarios

In theory, GitOps looks clean and deterministic:

> Git = Source of Truth → Cluster = Desired State

But in real Kubernetes clusters, things are never that clean.

Clusters are dynamic, and multiple actors modify state outside Git:

* Humans (`kubectl edit`)
* Kubernetes controllers (admission controllers, autoscalers)
* Legacy resources already running in the cluster

This is where **production-grade GitOps becomes complex and interesting**.

We now study three critical real-world scenarios.

### Scenario 1: Manual Drift (Human Changes in Cluster)

### Situation

An administrator manually modifies a running deployment:

```bash
kubectl edit deployment app
```

They change something like:

* replicas (from 3 to 5)
* image tag
* resource limits

But this change is **not reflected in Git**.

### What Argo CD sees

Argo CD continuously compares:

```
Git Desired State ≠ Live Cluster State
```

This mismatch is called:

> Configuration Drift

### What happens next?

Argo CD detects drift and triggers reconciliation:

### Default Behavior

* Cluster is reverted back to Git state
* Manual changes are overwritten
* History shows the drift event

### Important Exception

Some teams configure:

| Option                 | Behavior                 |
|------------------------|--------------------------|
| auto-sync disabled     | Requires manual approval |
| selective ignore rules | Certain fields ignored   |
| sync waves             | Sequential deployment    |

So behavior can vary in controlled environments.

### Key Insight

GitOps assumes:

> ❗ Git is always the authority, not humans operating directly on the cluster

### Scenario 2: Admission Controllers Mutating Resources

### Situation

Kubernetes itself modifies resources during admission.

**Example:**

You define a Pod in Git:

```yaml
resources: {}
```

But an admission controller injects:

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "256Mi"
```

This is common with:

* Pod Security Policies / Admission Webhooks
* Resource quota injectors
* Service mesh sidecars (Istio, Linkerd)
* Security policies (OPA Gatekeeper)
* Istio proxy injection

### Key Question

> Will GitOps remove these injected fields?

### Answer: It depends on diff strategy

Argo CD compares:

* **Git desired state**
* **Live cluster state**

But it can be configured using:

#### 1. Ignore Differences

You can tell Argo CD:

> "Do NOT treat these fields as drift"

**Example Configuration:**

```yaml
spec:
  ignoreDifferences:
  - group: ""
    jsonPointers:
    - /spec/template/spec/initContainers
    kind: Pod
```

Commonly ignored fields:

| Field                        | Reason                   |
|------------------------------|--------------------------|
| sidecar.istio.io/status      | Istio injection          |
| metadata.annotations.kubectl | Kubectl annotation       |
| spec.initContainers          | Init container injection |

#### 2. Strict Reconciliation (Not Recommended)

If NOT ignored:

* Argo CD will continuously overwrite injected values
* Causing a "reconciliation loop"
* Wasting cluster resources

### Production Reality

This is one of the most common real-world GitOps challenges:

> GitOps must coexist with Kubernetes controllers, not fight them.

### Key Insight

Admission controllers introduce:

> ⚠️ "legitimate mutations outside Git"

So GitOps must be carefully tuned with `ignore` configurations.

### Scenario 3: Existing Cluster Resources (Legacy Systems)

### Situation

You adopt GitOps in a cluster that already has:

* running applications
* manually deployed services
* legacy workloads

These were **not created from Git**.

### Key Question

> "What happens to these existing resources?"

Will Argo CD:

* delete them?
* overwrite them?
* ignore them?

### Answer: Nothing happens automatically

Argo CD does NOT assume control blindly.

Instead, resources must be explicitly handled.

### Three Possible Strategies

#### 1. Adopt (Import into GitOps)

You:

* Export existing YAML from cluster
* Add to Git repository
* Create Argo CD Application pointing to Git

Now it becomes managed by GitOps.

**Best for:** Production migration to GitOps

#### 2. Ignore (Leave Unmanaged)

* Argo CD ignores existing resources
* They remain untouched
* No automatic synchronization

**Best for:** Gradual GitOps adoption

#### 3. Replace (Recreate from Git)

* Delete existing resources
* Recreate from Git definitions

> ⚠️ **Warning:** May cause downtime

**Best for:** Standardization efforts

### Scenario Summary Table

| Scenario                     | Default GitOps Behavior       | Configurable? |
|------------------------------|-------------------------------|---------------|
| Manual Drift                 | Reconciles back to Git        | Yes           |
| Admission Controller Changes | May be overwritten or ignored | Yes           |
| Existing Resources           | Ignored unless adopted        | Yes           |

### Mini Summary

Production GitOps is not just about syncing Git to Kubernetes.

It must handle:

* Human changes (drift)
* System-generated mutations (admission controllers)
* Legacy infrastructure onboarding

> Real-world GitOps is a **conflict resolution system**, not just a deployment tool.

## 10. Argo CD vs Flux CD

### Comparison Overview

Both Argo CD and Flux CD are CNCF Graduated GitOps tools.

However, they have different design philosophies and strengths.

### Detailed Comparison

| Feature               | Argo CD                 | Flux CD v2                |
|-----------------------|-------------------------|---------------------------|
| **Architecture**      | Integrated (monolithic) | Modular (GitOps Toolkit)  |
| **User Interface**    | Rich built-in UI        | Minimal (web UI optional) |
| **CLI**               | argocd CLI              | flux CLI                  |
| **Learning Curve**    | Easier                  | Moderate                  |
| **Authentication**    | Built-in (Dex)          | External (OAuth2 Proxy)   |
| **Multi-tenancy**     | Built-in                | via GitOps Toolkit        |
| **ApplicationSet**    | Yes                     | Yes (via generators)      |
| **Helm Support**      | Native                  | Native                    |
| **Kustomize Support** | Native                  | Native                    |
| **Extensibility**     | Plugins                 | CRD-based                 |

### Architecture Differences

#### Argo CD

```
┌────────────────────────────────────┐
│            Argo CD                 │
├────────────────────────────────────┤
│ API Server ← UI/CLI                │
│ Application Controller             │
│ Repo Server (Git)                  │
│ Redis (Cache)                      │
│ Dex (Auth)                         │
└────────────────────────────────────┘
```

> **Philosophy:** All-in-one integrated solution

#### Flux CD v2

```
┌───────────────────────────────────────┐
│         Flux (GitOps Toolkit)         │
├───────────────────────────────────────┤
│ Source Controller → Git watching      │
│ Kustomize Controller → Reconciliation │
│ Helm Controller → Helm releases       │
│ Notification Controller → Alerts      │
│ Image Updater → Auto-update           │
└───────────────────────────────────────┘
```

> **Philosophy:** Modular, composable components

### When to Choose Argo CD

* Rich UI is important
* Fast onboarding needed
* Enterprise SSO required
* Team prefers integrated solution
* NeedApplicationSet features

### When to Choose Flux CD

* Need modular architecture
* Prefer GitOps Toolkit pattern
* Custom automation needed
* Want fine-grained control
* Building custom controllers

## 11. Final Summary

### Recap

GitOps architecture is far more than:

> "Deploy YAML from Git."

It is a sophisticated distributed control system built around:

* State management
* Reconciliation
* Automation
* Self-healing
* Authentication
* Continuous synchronization

Argo CD demonstrates how these principles are implemented using specialized microservices:

| Component                 | Responsibility            |
|---------------------------|---------------------------|
| Repo Server               | Git interface             |
| Application Controller    | Reconciliation engine     |
| API Server                | Human interaction         |
| ApplicationSet Controller | Fleet management          |
| Notifications Controller  | Alerts                    |
| Dex                       | Enterprise authentication |
| Redis                     | Caching and state         |

Once you understand this architecture, switching to tools like Flux CD becomes much easier.

### Key Insight

> "Real-world GitOps is a **conflict resolution system**, not just a deployment tool."

You must handle:

* Manual drift from humans
* Mutations from admission controllers
* Legacy infrastructure onboarding

## 12. Key Takeaways

- GitOps uses Git as the single source of truth
- Continuous reconciliation is the core mechanism
- Argo CD uses multiple microservices working together
- Repo Server reads Git (desired state)
- Application Controller reconciles state (actual vs desired)
- API Server enables user access (UI + CLI)
- Dex handles enterprise SSO
- Redis stores cache and state
- ApplicationSet handles fleet deployments
- Notifications keep teams informed
- GitOps handles drift automatically
- Real-world clusters introduce advanced reconciliation challenges
- Admission controller mutations require ignore configurations
- Legacy resources need explicit adoption strategy
- Both Argo CD and Flux v2 are production-ready CNCF Graduated projects

## 13. Interview Questions

> ### Basic Questions

#### Q1: What is GitOps?

**Answer:** GitOps is a declarative operational model where Git contains the desired state of the system, and automated controllers continuously reconcile actual state with desired state.

#### Q2: Why is Git the single source of truth?

**Answer:** Because:

* Version-controlled (audit trail)
* Immutable history
* Code review capabilities
* Standard developer workflow
* Enables rollback

#### Q3: What is reconciliation in GitOps?

**Answer:** The continuous process of comparing desired state (Git) with actual state (cluster) and correcting any drift automatically.

#### Q4: What are the core components of Argo CD?

**Answer:**

* Repo Server (Git interface)
* Application Controller (reconciliation)
* API Server (UI/CLI)
* Dex (SSO)
* Redis (cache)

#### Q5: What is the difference between Argo CD and Flux CD?

**Answer:** Argo CD is an integrated solution with rich UI, while Flux CD v2 is modular and composed of multiple controllers (GitOps Toolkit).

> ### Intermediate Questions

#### Q6: How does Argo CD detect drift?

**Answer:**

1. Application Controller reads desired state from Repo Server
2. Queries Kubernetes API for actual state
3. Compares desired vs actual using diff logic
4. If mismatch found → triggers sync

#### Q7: How does GitOps handle admission controller mutations?

**Answer:** Using `ignoreDifferences` configuration in Application or Project-level settings. This tells Argo CD to ignore specific fields during comparison.

#### Q8: What happens when someone manually edits a Kubernetes resource managed by GitOps?

**Answer:** Argo CD detects drift in next reconciliation and by default reconciles back to Git state. Can be configured to require manual approval.

#### Q9: How does Argo CD authenticate users?

**Answer:** Through API Server integrated with Dex, which supports OIDC, OAuth, and external identity providers (GitHub, Google, LDAP, etc.)

#### Q10: What is ApplicationSet controller?

**Answer:** A controller that generates multiple Argo CD Applications from templates, enabling fleet-scale deployments and cluster management.

> ### Advanced Questions

#### Q11: How would you handle a legacy cluster migration to GitOps?

**Answer:**

1. Inventory existing resources
2. Export current YAML
3. Commit to Git repository
4. Create Argo CD Application
5. Choose strategy: Adopt, Ignore, or Replace
6. Monitor and validate
7. Gradually migrate workloads

#### Q12: Explain the controller reconciliation loop in detail.

**Answer:**

1. **Observe:** Get desired state from Git via Repo Server
2. **Observe:** Get actual state from Kubernetes API
3. **Compare:** Detect drift using diff algorithm
4. **Correct:** Apply changes if drift detected
5. **Repeat:** Schedule next reconciliation cycle

#### Q13: How do you secure GitOps in production?

**Answer:**

* Use RBAC for fine-grained access
* Enable SSO (Dex)
* Implement Git branch protection
* Use Git signed commits
* Apply network policies
* Enable audit logging
* Use external secrets (Vault, Sealed Secrets)

## 14. What Comes Next

### Recommended Next Steps

The next learning stage includes:

| Topic                        | Description                                      |
|------------------------------|--------------------------------------------------|
| **Advanced Argo CD**         | ApplicationSets, multi-tenancy                   |
| **Flux CD Deep Dive**        | GitOps Toolkit, modular architecture             |
| **Secrets Management**       | Vault, Sealed Secrets, External Secrets Operator |
| **Multi-cluster Management** | Deploying across multiple clusters               |
| **GitOps at Scale**          | Handling hundreds of applications                |
| **Security Hardening**       | RBAC, audit logging, network policies            |
| **Observability**            | Metrics, logging, alerting                       |

### Additional Resources

| Resource                  | URL                                     |
|---------------------------|-----------------------------------------|
| Argo CD Documentation     | https://argo-cd.readthedocs.io/         |
| Flux CD Documentation     | https://fluxcd.io/flux/                 |
| CNCF GitOps Working Group | https://gitops.openshift.io/            |
| Weave GitOps              | https://www.weave.works/product/gitops/ |
