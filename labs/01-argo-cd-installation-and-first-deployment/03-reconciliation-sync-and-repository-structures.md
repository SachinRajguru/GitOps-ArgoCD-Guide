
## 📄 `01-argo-cd-installation-and-first-deployment`

## Section 3: Continuous Reconciliation, Drift Detection, Sync Status, Application Health, and Git Repository Structures (Plain YAML, Helm, and Kustomize)

### Learning Objectives

After completing this section, you will be able to:

* Understand Continuous Reconciliation
* Understand Desired State vs Actual State
* Understand Drift Detection
* Understand Sync Status
* Understand Application Health Status
* Learn how Argo CD reacts to Git changes
* Understand Self-Healing concepts
* Learn why Argo CD is called "Not Opinionated"
* Deploy applications using:
  * Plain Kubernetes Manifests
  * Helm Charts
  * Kustomize
* Understand how organizations structure Git repositories

## 20. The Real Power of Argo CD

At this point, many engineers ask:

> Why should I use Argo CD?

After all, the same deployment could have been done using:

```bash
kubectl apply -f deployment.yaml
```

or

```bash
helm install
```

So what exactly makes Argo CD special?

The answer is:

```text
Continuous Reconciliation
```

This is the heart of GitOps.

### Traditional Deployment Problem

Consider a deployment:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
spec:
  replicas: 3
```

A CI/CD pipeline deploys it.

Everything looks fine.

A week later, an administrator runs:

```bash
kubectl edit deployment guestbook
```

and changes:

```yaml
spec:
  replicas: 5
```

Now:

Git says:

```yaml
replicas: 3
```

Cluster says:

```yaml
replicas: 5
```

Question:

Who notices this difference?

In traditional CI/CD:

```text
Nobody
```

The pipeline already finished.

> The deployment is no longer monitored.

### The GitOps Solution

GitOps introduces:

```text
Continuous Monitoring
```

instead of

```text
One-Time Deployment
```

### Understanding Desired State vs Actual State

This is the most important concept in GitOps.

#### Desired State

Desired State means:

```text
What Git Says
```

Example:

```yaml
replicas: 3
image: nginx:1.25
```

Git becomes the source of truth.

#### Actual State

Actual State means:

```text
What Kubernetes Currently Has
```

Example:

```yaml
replicas: 5
image: nginx:1.25
```

### Visualization

```text
Git Repository
     │
     ▼
Desired State

replicas: 3
```

```text
Kubernetes Cluster
     │
     ▼
Actual State

replicas: 5
```

Mismatch detected.

### What Happens Next?

Argo CD continuously compares:

```text
Desired State
VS
Actual State
```

When differences appear:

```text
Out Of Sync
```

is reported.

### Real-World Analogy

Imagine a school attendance register.

Official register:

```text
30 Students
```

Actual classroom:

```text
28 Students
```

The teacher immediately notices the mismatch.

Argo CD behaves similarly.

It continuously checks whether reality matches the official record.

## 21. Understanding Reconciliation

Reconciliation means:

```text
Continuously comparing
and correcting differences
```

between:

```text
Git
and
Kubernetes
```

### Reconciliation Loop

The Application Controller repeatedly executes:

| Step | Action             |
|------|--------------------|
| 1    | Read Git           |
| 2    | Read Cluster       |
| 3    | Compare            |
| 4    | Detect Differences |
| 5    | Take Action        |
| 6    | Repeat Forever     |

### Architecture View

```text
                Git Repository
                       │
                       ▼
             Desired Configuration
                       │
                       ▼
             Application Controller
                       │
                Compare States
                       │
          ┌────────────┴────────────┐
          │                         │
          ▼                         ▼
    States Match              States Differ
          │                         │
          ▼                         ▼
       Synced                  Out Of Sync
```

### Why Reconciliation Matters

Without reconciliation:

```text
Deployment Success
      ≠
Current Reality
```

A deployment may have succeeded months ago.

Nobody knows what happened afterward.

GitOps eliminates this uncertainty.

> Key Point: Reconciliation is not a one-time action. It runs continuously in a loop.

## 22. Understanding Sync Status

Every Argo CD application has a Sync Status.

This tells us:

```text
Does Kubernetes match Git?
```

### Synced

Meaning:

```text
Git = Cluster
```

Example:

| Source  | replicas | Status     |
|---------|----------|------------|
| Git     | 3        |            |
| Cluster | 3        |            |
|         |          | **Synced** |

### Out Of Sync

Meaning:

```text 
Git ≠ Cluster
```

Example:

| Source  | replicas | Status          |
|---------|----------|-----------------|
| Git     | 3        |                 |
| Cluster | 5        |                 |
|         |          | **Out Of Sync** |

### Visualization

```text
Git Repository
      │
      ▼
 replicas = 3

      │
      ▼

Kubernetes Cluster
      │
      ▼
 replicas = 5

Status:
OutOfSync
```

### How Argo CD Displays This

Inside the UI:

```text
Application
   │
   ├── Health Status
   └── Sync Status
```

Both are different concepts.

Many beginners confuse them.

## 23. Understanding Health Status

Sync Status answers:

```text
Does Git match Kubernetes?
```

Health Status answers:

```text
Is the application working?
```

### Example 1: Healthy and Synced

| State            | Value   |
|------------------|---------|
| Git replicas     | 3       |
| Cluster replicas | 3       |
| Pod Status       | Running |

**Result:**

| Status      | Value   |
|-------------|---------|
| Sync Status | Synced  |
| Health      | Healthy |

### Example 2: Synced but Degraded

| State            | Value            |
|------------------|------------------|
| Git replicas     | 3                |
| Cluster replicas | 3                |
| Pod Status       | CrashLoopBackOff |

**Result:**

| Status      | Value    |
|-------------|----------|
| Sync Status | Synced   |
| Health      | Degraded |

> Notice: Git matches Kubernetes, but the application is broken.

### Example 3: OutOfSync but Healthy

| State            | Value   |
|------------------|---------|
| Git replicas     | 3       |
| Cluster replicas | 5       |
| Pod Status       | Running |

**Result:**

| Status      | Value     |
|-------------|-----------|
| Sync Status | OutOfSync |
| Health      | Healthy   |

This is a common interview concept.

### Interview Question

Can an application be:

```text
Healthy
and
Out Of Sync
```

Answer:

Yes.

The application can work perfectly while still differing from Git.

Health and Sync are independent in Argo CD:

* Sync Status → compares Git vs desired cluster state
* Health Status → evaluates actual runtime behavior

So an application can be:

* Running correctly (Healthy)
* But still different from Git (OutOfSync)

### Health States

Common statuses:

```text
Healthy
```

```text
Progressing
```

```text
Missing
```

```text
Suspended
```

```text
Degraded
```

| Health Status | Meaning |
|---------------|---------|
| `Healthy` | Application is running correctly |
| `Progressing` | Application is being deployed or upgraded |
| `Missing` | Expected resources are not found |
| `Suspended` | Deployment is paused |
| `Degraded` | Application is not functioning correctly |

## 24. What Happens When Git Changes?

Let's examine a real scenario.

Current deployment (Initial State):

| Source  | replicas |
|---------|----------|
| Git     | 1        |
| Cluster | 1        |

Everything is synchronized.

Now a developer updates Git:

```yaml
# deployment.yaml
spec:
  replicas: 3
```

and pushes:

```bash
git push origin main
```

### What Happens Next?

Application Controller detects:

```text
Git Changed
```

Argo CD executes:

| Step | Action                |
|------|-----------------------|
| 1    | Read Repository       |
| 2    | Generate Manifest     |
| 3    | Compare Current State |
| 4    | Detect Difference     |
| 5    | Apply Changes         |

Result:

```text
replicas: 3
```

appears automatically inside Kubernetes.

### Visualization

```text
Developer
    │
    ▼
Git Push
    │
    ▼
Git Repository
    │
    ▼
Repo Server
    │
    ▼
Application Controller
    │
    ▼
Kubernetes Cluster
```

### This Is True GitOps

No engineer manually runs:

```bash
kubectl apply
```

Everything is driven by Git.

## 25. Understanding Drift Detection

Drift means:

```text
Cluster State
Has Drifted Away
From Git State
```

### Example

Git:

```yaml
replicas: 2
```

Administrator executes:

```bash
kubectl scale deployment guestbook \
--replicas=10
```

Now:

Git:

```yaml
replicas: 2
```

Cluster:

```yaml
replicas: 10
```

Argo CD detects:

```text
Configuration Drift
```

### Why Drift Detection Is Important

> Without GitOps:

Manual changes remain unnoticed.

> With GitOps:

Manual changes become visible immediately.

### Enterprise Example

Production environment.

An engineer manually updates:

```yaml
image: nginx:latest
```

instead of:

```yaml
image: nginx:1.25
```

This creates inconsistency.

Argo CD reports it instantly.

## 26. Why Argo CD Is Called "Not Opinionated"

> One of Argo CD's strongest features is flexibility.

Argo CD does not force you into a specific repository structure or configuration management approach.

### What Does Opinionated Mean?

An opinionated tool says:

```text
Use This Way Only
```

For example, some platforms require:

* A specific folder structure
* Specific naming conventions
* Specific Templates

Argo CD does not.

Instead it says:

```text
Store Manifests
In Whatever Format
You Prefer
```

### Supported Formats

Argo CD supports:

```text
Plain YAML
```

```text
Helm
```

```text
Kustomize
```

```text
Plugins
```

This flexibility is one of the reasons Argo CD is widely adopted.

| Format     | Description                    |
|------------|--------------------------------|
| Plain YAML | Standard Kubernetes manifests  |
| Helm       | HHelm charts and templates     |
| Kustomize  | Kustomize bases and overlay    |
| Plugins    | Custom plugin-based generation |

### Why This Matters

Different teams can adopt different Git repository structures and deployment approaches without changing Argo CD itself.

For example:

* Team A may use plain Kubernetes YAML files
* Team B may use Helm charts
* Team C may use Kustomize overlays

All can be managed by the same Argo CD installation.

## 27. Plain Manifest Structure

The Guestbook example uses Plain YAML.

Repository:

```text
guestbook/
│
├── deployment.yaml
└── service.yaml
```

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
...
```

Service:

```yaml
apiVersion: v1
kind: Service
...
```

| Advantages         | Disadvantages                        |
|--------------------|--------------------------------------|
| Easy to understand | Harder to manage large environments  |
| Simple             | Duplicate configurations across apps |
| Beginner friendly  | No parameterization                  |

### When to Use Plain YAML

* Learning Argo CD
* Small projects
* Simple applications with few resources

## 28. Helm-Based Structure

Many organizations use Helm.

Helm introduces templating.

Structure:

```text
helm-guestbook/
│
├── Chart.yaml
├── values.yaml
└── templates/
      │
      ├── deployment.yaml
      └── service.yaml
```

### Why Helm?

Instead of hardcoding values directly in Kubernetes manifests:

```yaml
replicas: 3
```

Helm allows you to use placeholders in templates:

```yaml
# templates/deployment.yaml
spec:
  replicas: {{ .Values.replicas }}
```

The actual value is defined in:

```yaml
# values.yaml
replicas: 3
```

During chart rendering, Helm replaces the placeholder with the corresponding value from `values.yaml`.

This separation of templates and values allows the same chart to be reused across multiple environments (development, staging, and production) by changing only the values while keeping the templates unchanged.

### Real-World Example

**Environment-Specific Values**

Development:

```yaml
# values-dev.yaml
replicas: 1
image: nginx:dev
```

Production:

```yaml
# values-prod.yaml
replicas: 10
image: nginx:1.25
```

The same Helm chart can be deployed to different environments using different values files.

As a result:

* Templates remain unchanged
* Environment-specific settings stay separate
* Configuration management becomes easier and more consistent

### Benefits

| Benefit              | Description                  |
|----------------------|------------------------------|
| Reusability          | One chart deploys everywhere |
| Parameterization     | Customize via values files   |
| Environment-specific | Dev, Staging, Prod configs   |

### When to Use Helm

* Medium to large teams
* Multiple environments
* Complex configurations

## 29. Kustomize-Based Structure

Another popular approach.

Structure:

```text
kustomize-guestbook/
│
├── base/
│    ├── deployment.yaml
│    ├── service.yaml
│    └── kustomization.yaml
│
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── config.yaml
    │
    └── prod/
        ├── kustomization.yaml
        └── config.yaml
```

### Concept

Base:

```text
Common Configuration
```

Overlay:

```text
Environment Customizations
```

### Example

**Base Configuration**

```yaml
# base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
spec:
  replicas: 1
```

**Overlay Configuration**

Production overlay:

```yaml
# overlays/prod/config.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
spec:
  replicas: 5
```

Benefits

| Benefit           | Description             |
|-------------------|-------------------------|
| Native Kubernetes | No extra tooling        |
| No templating     | Plain YAML with patches |
| Easy debugging    | See final result easily |

### When to Use Kustomize

* Complex multi-environment deployments
* Teams familiar with Kubernetes
* Avoiding template languages


### Which One Should You Use?

There is no universal answer.

| Approach   | Best For                                     |
|------------|----------------------------------------------|
| Plain YAML | Learning, small projects                     |
| Helm       | Medium to large teams, multiple environments |
| Kustomize  | Kubernetes-native workflows                  |

becomes attractive.

### Why Argo CD Doesn't Care

Argo CD simply asks:

```text
Can I Generate Kubernetes Manifests?
```

If yes:

It deploys them.

### Key Learning

Argo CD focuses on:

```text
Delivery
```

not

```text
Manifest Design
```

The design choice belongs to the organization.

## Section Summary

In this section we:

* ✓ Learned Continuous Reconciliation
* ✓ Understood Desired State vs Actual State
* ✓ Learned Drift Detection
* ✓ Understood Sync Status (Synced vs OutOfSync)
* ✓ Understood Health Status (Healthy, Degraded, Progressing)
* ✓ Compared Healthy vs Synced states simultaneously
* ✓ Learned how Git changes trigger automatic deployments
* ✓ Understood GitOps workflows
* ✓ Understood why Argo CD is not opinionated
* ✓ Explored Plain YAML repositories
* ✓ Explored Helm repositories
* ✓ Explored Kustomize repositories
* ✓ Learned how organizations structure Git repositories
* ✓ Compared different manifest approaches

## Key Takeaways

1. **Continuous Reconciliation** is the foundation of GitOps — it runs forever, not once.
2. **Desired State** comes from Git — Git is the source of truth.
3. **Actual State** comes from Kubernetes — what's currently running.
4. **Drift** occurs when Git and Kubernetes differ.
5. **Sync Status** measures configuration consistency (Synced vs OutOfSync).
6. **Health Status** measures application functionality (Healthy, Degraded, Progressing).
7. **Healthy and OutOfSync** can occur simultaneously.
8. Argo CD continuously monitors Git repositories.
9. Argo CD supports **Plain YAML**, **Helm**, and **Kustomize**.
10. Argo CD focuses on **delivery**, not repository structure.

## Interview Questions

#### 1. What is Continuous Reconciliation?

**Answer:** The process of continuously comparing desired state in Git with actual state in Kubernetes and automatically correcting differences. It runs in an infinite loop, not as a one-time action.

#### 2. What is Configuration Drift?

**Answer:** A situation where Kubernetes cluster configuration differs from the configuration stored in Git. This can happen through manual changes or unintended modifications.

#### 3. What is Desired State?

**Answer:** The intended configuration stored in Git. This is what the application SHOULD look like according to the source of truth.

#### 4. What is Actual State?

**Answer:** The configuration currently running inside Kubernetes. This is what the application ACTUALLY looks like right now.

#### 5. What does Sync Status indicate?

**Answer:** Whether Git and Kubernetes configurations match. If they match, status is "Synced". If they differ, status is "OutOfSync".

#### 6. What does Health Status indicate?

**Answer:** Whether the deployed application is functioning correctly. States include Healthy, Progressing, Degraded, Missing, and Suspended.

#### 7. Can an application be Healthy but OutOfSync?

**Answer:** Yes. The application can be running perfectly while still differing from Git. For example, someone manually scaled replicas from 3 to 10 — pods are running (Healthy) but cluster differs from Git (OutOfSync).

#### 8. Why is Argo CD called non-opinionated?

**Answer:** Because it supports multiple manifest formats (Plain YAML, Helm, Kustomize) and does not force a specific repository structure. It focuses on delivery, not how you design manifests

#### 9. Which component performs reconciliation?

**Answer:** pplication Controller (`argocd-application-controller`). It continuously monitors Git and Kubernetes, compares states, and triggers sync when needed.

#### 10. What repository structures are supported by Argo CD?

**Answer:** Plain YAML, Helm Charts, Kustomize, and custom plugin-based approaches.

| Format     | Tool            |
|------------|-----------------|
| Plain YAML | kubectl, manual |
| Helm       | Helm CLI        |
| Kustomize  | Kustomize CLI   |
| Plugins    | Custom tools    |

## End of Section 3

**Next section:** *Argo CD CLI Deep Dive — Installation, Authentication, Application Creation, Sync Operations, Managing Applications from the Command Line, and UI vs CLI Comparison.*
