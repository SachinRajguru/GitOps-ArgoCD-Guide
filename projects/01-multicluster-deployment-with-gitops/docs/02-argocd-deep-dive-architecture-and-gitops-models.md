
## 📄 GitOps Multi-Cluster Deployment with Argo CD (Hub–Spoke Model on AWS EKS)

## Section 2 — Argo CD Deep Dive: Architecture and GitOps Workflow

## 3. Understanding Argo CD (Deep Dive)

### Learning Objectives

By the end of this section, you will understand:

* What Argo CD is internally
* How Argo CD works as a Kubernetes controller
* Core components of Argo CD architecture
* How it fits into GitOps workflow

### 3.1 What is Argo CD?

> Argo CD is a **Kubernetes-native continuous delivery controller** that continuously watches Git repositories and ensures the cluster matches the desired state defined in Git.

#### Simple Interpretation:

* Git = Desired state
* Kubernetes cluster = Actual state
* Argo CD = Reconciler (keeps both equal)

### 3.2 Why Argo CD Exists

Traditional CD tools:

* Jenkins
* Ansible
* Shell scripts

❌ Problems:

* Imperative deployment
* No continuous reconciliation
* No drift detection
* No multi-cluster awareness

✅ Argo CD solves this by being:

* Declarative
* Continuous
* Self-healing
* Git-driven

### 3.3 Argo CD Architecture

#### Core Components

```bash
                ┌────────────────────┐
                │   Git Repository   │
                └─────────┬──────────┘
                          │
                          ▼
        ┌────────────────────────────────────┐
        │              Argo CD               │
        │                                    │
        │  ┌──────────────────────────────┐  │
        │  │ API Server                   │  │
        │  │ (UI + CLI + REST API)        │  │
        │  ├──────────────────────────────┤  │
        │  │ Repository Server            │  │
        │  ├──────────────────────────────┤  │
        │  │ Application Controller       │  │
        │  ├──────────────────────────────┤  │
        │  │ Dex (Auth)                   │  │
        │  ├──────────────────────────────┤  │
        │  │ Redis (Cache)                │  │
        │  └──────────────────────────────┘  │
        └────────────────────────────────────┘
                          │
                          ▼
                ┌─────────────────────┐
                │ Kubernetes Clusters │
                └─────────────────────┘
```

### 3.4 Component Breakdown

#### 1. API Server

* UI backend
* CLI communication layer
* REST API provider

➤ You interact with Argo CD through this

#### 2. Repository Server

* Clones Git repositories
* Parses manifests (YAML/Helm/Kustomize)
* Prepares deployment objects

#### 3. Application Controller

* Heart of Argo CD
* Continuously compares Git vs Cluster state
* Performs reconciliation

➤ This is the “brain”

#### 4. Dex (Authentication)

* Handles login (SSO, OAuth, LDAP)
* Manages identity providers

#### 5. Redis

* Caches cluster state
* Improves performance

### 3.5 Argo CD Workflow

```id="workflow1"
Git Commit → Repository Server → Controller Compare → Sync → Kubernetes Cluster
```

#### Analogy

Argo CD is like:

> A strict librarian who keeps checking books (cluster state) against the catalog (Git) and fixes any mismatch automatically.

### 3.6 Key Features of Argo CD

* GitOps-based deployment
* Multi-cluster support
* Automatic drift detection
* Self-healing
* Web UI dashboard
* CLI support
* RBAC security

## 4. Why Organizations Use Multi-Cluster Kubernetes

### 4.1 Reality in Enterprises

Organizations DO NOT use one cluster.

They use multiple clusters for:

#### Environments

* Dev
* QA
* Staging
* Production

#### Feature Isolation

* Feature-1 cluster
* Feature-2 cluster

#### Team Isolation

* Team A cluster
* Team B cluster

### 4.2 Why Not One Cluster?

#### ❌ Problems with single cluster:

* Resource contention
* Security risks
* No isolation
* Difficult debugging
* Scaling limitations

### 4.3 Real-World Example

Imagine:

* 50 microservices
* 10 teams
* 5 environments

➤ One cluster becomes unmanageable

### 4.4 Multi-Cluster Strategy

Organizations split workloads:

* Development cluster → unstable testing
* QA cluster → integration testing
* Production cluster → stable users
* Feature clusters → experimental work

### 4.5 Key Insight

Developers may request:

> “Give us a new cluster for Feature X”

Because:

* They don't want interference from other teams
* They want isolation

#### Analogy

A Kubernetes cluster is like:

> A city

Multi-cluster system is like:

> Multiple cities for different purposes (industrial, residential, experimental)

### 4.6 Deployment Challenge

Now consider:

You have:

* Dev cluster
* QA cluster
* Feature cluster 1
* Feature cluster 2

And application:

> Guestbook v1 → v2 upgrade

#### ❌ Traditional Problem

You must:

* Login to each cluster
* Run kubectl apply
* Or run CI/CD scripts repeatedly

Problems:

* Manual effort
* Human errors
* No consistency guarantee

#### ✅ GitOps Solution (Argo CD)

Instead:

* Push change once to Git
* Argo CD propagates to all clusters

### 4.7 Hub–Spoke Requirement

To manage multiple clusters efficiently, we introduce:

> Central control cluster (Hub) + Worker clusters (Spokes)

### Interview Questions

#### Q1: What are the main components of Argo CD?

**A:** API Server, Repository Server, Application Controller, Dex, Redis.

#### Q2: Why do enterprises use multiple Kubernetes clusters?

**A:** For isolation, security, scalability, and environment separation.

#### Q3: What is the role of the Application Controller?

**A:** It continuously reconciles Git state with cluster state.

#### Q4: What problem does multi-cluster architecture solve?

**A:** It avoids resource conflicts and enables environment isolation.

### Summary

* Argo CD is a Kubernetes-native GitOps controller
* It continuously reconciles cluster state with Git
* It consists of API server, repo server, controller, Dex, Redis
* Enterprises use multiple clusters for isolation and scaling
* Multi-cluster setups require centralized management
* This leads to Hub–Spoke architecture

## ➤ Next: Section 3

We will cover:

* Hub–Spoke Architecture (Deep Dive)
* Standalone Architecture
* Comparison Between Both Models
* When to Use Each Model
* Real Enterprise Scenarios
