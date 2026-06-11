
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

The deployment is no longer monitored.

### The GitOps Solution

GitOps introduces:

```text
Continuous Monitoring
```

instead of

```text
One-Time Deployment
```

### Desired State vs Actual State

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

```text
Step 1:
Read Git
```

↓

```text
Step 2:
Read Cluster
```

↓

```text
Step 3:
Compare
```

↓

```text
Step 4:
Detect Differences
```

↓

```text
Step 5:
Take Action
```

↓

Repeat Forever

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
    States Match            States Differ
          │                         │
          ▼                         ▼
      Synced                 Out Of Sync
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

Git:

```yaml
replicas: 3
```

Cluster:

```yaml
replicas: 3
```

Status:

```text
Synced
```

### Out Of Sync

Meaning:

```text
Git ≠ Cluster
```

Example:

Git:

```yaml
replicas: 3
```

Cluster:

```yaml
replicas: 5
```

Status:

```text
Out Of Sync
```

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

### Example 1

Git:

```yaml
replicas: 3
```

Cluster:

```yaml
replicas: 3
```

Pod:

```text
Running
```

Result:

```text
Sync Status:
Synced

Health:
Healthy
```

### Example 2

Git:

```yaml
replicas: 3
```

Cluster:

```yaml
replicas: 3
```

Pod:

```text
CrashLoopBackOff
```

Result:

```text
Sync Status:
Synced

Health:
Degraded
```

Notice:

Git matches Kubernetes.

But application is broken.

### Example 3

Git:

```yaml
replicas: 3
```

Cluster:

```yaml
replicas: 5
```

Pods:

```text
Running
```

Result:

```text
Sync Status:
OutOfSync

Health:
Healthy
```

This is an important interview question.

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

## 24. What Happens When Git Changes?

Let's examine a real scenario.

Current deployment:

```yaml
replicas: 1
```

Git:

```yaml
replicas: 1
```

Cluster:

```yaml
replicas: 1
```

Everything is synchronized.

Now a developer updates Git:

```yaml
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

```text
Read Repository
```

↓

```text
Generate Manifest
```

↓

```text
Compare Current State
```

↓

```text
Detect Difference
```

↓

```text
Apply Changes
```

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

Argo CD does not force you into a specific repository structure.

### What Does Opinionated Mean?

An opinionated tool says:

```text
Use This Way Only
```

Example:

Some platforms require:

```text
Specific Folder Structure
Specific Naming
Specific Templates
```

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

This flexibility makes Argo CD popular.

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

Advantages:

* Easy
* Simple
* Beginner Friendly

Disadvantages:

* Harder to manage large environments
* Duplicate configurations

### Real-World Usage

Commonly used for:

* Learning
* Small projects
* Simple applications

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

Instead of hardcoding values:

```yaml
replicas: 3
```

You write:

```yaml
replicas: {{ .Values.replicas }}
```

And define:

```yaml
replicas: 3
```

inside:

```text
values.yaml
```

### Benefits

* Reusability
* Parameterization
* Environment-specific configurations

### Real-World Example

Development:

```yaml
replicas: 1
```

Production:

```yaml
replicas: 10
```

Same chart.

Different values files.

## 29. Kustomize-Based Structure

Another popular approach.

Structure:

```text
base/
│
├── deployment.yaml
└── service.yaml

overlays/
│
├── dev
└── prod
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

Base:

```yaml
replicas: 1
```

Production overlay:

```yaml
replicas: 5
```

Advantages:

* Native Kubernetes approach
* No templating language
* Easy environment management

### Which One Should You Use?

There is no universal answer.

Small Team:

```text
Plain YAML
```

works fine.

Medium to Large Team:

```text
Helm
```

often preferred.

Complex Multi-Environment Deployments:

```text
Kustomize
```

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

## Chapter Summary

In this section we:

* ✓ Learned Continuous Reconciliation
* ✓ Understood Desired State vs Actual State
* ✓ Learned Drift Detection
* ✓ Understood Sync Status
* ✓ Understood Health Status
* ✓ Compared Healthy vs Synced states
* ✓ Learned how Git changes trigger deployments
* ✓ Learned GitOps workflows
* ✓ Understood why Argo CD is not opinionated
* ✓ Explored Plain YAML repositories
* ✓ Explored Helm repositories
* ✓ Explored Kustomize repositories
* ✓ Learned how organizations structure Git repositories

## Key Takeaways

1. Continuous Reconciliation is the foundation of GitOps.
2. Desired State comes from Git.
3. Actual State comes from Kubernetes.
4. Drift occurs when Git and Kubernetes differ.
5. Sync Status measures configuration consistency.
6. Health Status measures application health.
7. Healthy and OutOfSync can occur simultaneously.
8. Argo CD continuously monitors Git repositories.
9. Argo CD supports Plain YAML, Helm, and Kustomize.
10. Argo CD focuses on delivery, not repository structure.

## Interview Questions

#### 1. What is Continuous Reconciliation?

**Answer:** The process of continuously comparing desired state in Git with actual state in Kubernetes and correcting differences.

#### 2. What is Configuration Drift?

**Answer:** A situation where Kubernetes configuration differs from the configuration stored in Git.

#### 3. What is Desired State?

**Answer:** The intended configuration stored in Git.

#### 4. What is Actual State?

**Answer:** The configuration currently running inside Kubernetes.

#### 5. What does Sync Status indicate?

**Answer:** Whether Git and Kubernetes configurations match.

#### 6. What does Health Status indicate?

**Answer:** Whether the deployed application is functioning correctly.

#### 7. Can an application be Healthy but OutOfSync?

**Answer:** Yes.

#### 8. Why is Argo CD called non-opinionated?

**Answer:** Because it supports multiple manifest formats and does not force a specific repository structure.

#### 9. Which component performs reconciliation?

**Answer:** Application Controller.

#### 10. What repository structures are supported by Argo CD?

**Answer:** Plain YAML, Helm Charts, Kustomize, and custom plugin-based approaches.

## End of Section 3

**Next section:** *Argo CD CLI Deep Dive — Installation, Authentication, Application Creation, Sync Operations, Managing Applications from the Command Line, and UI vs CLI Comparison.*
