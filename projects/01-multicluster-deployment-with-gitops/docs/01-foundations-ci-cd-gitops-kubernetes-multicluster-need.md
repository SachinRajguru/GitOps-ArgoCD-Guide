
## 📄 GitOps Multi-Cluster Deployment with Argo CD (Hub–Spoke Model on AWS EKS)

## Section 1 — Foundations: CI/CD, GitOps, and the Need for Multi-Cluster Kubernetes

## 1. Introduction to Multi-Cluster Deployment

### Learning Objectives

By the end of this section, you will understand:

* What multi-cluster Kubernetes deployment means
* Why organizations don’t use a single cluster
* How Argo CD fits into modern delivery pipelines
* Why GitOps replaces traditional CD methods

### 1.1 Introduction

In modern DevOps workflows, we deploy applications to **multiple Kubernetes clusters simultaneously** using **Argo CD**, a GitOps-based continuous delivery tool.

Argo CD is widely used because it solves major problems in traditional deployment pipelines.

### 1.2 Key Definition — Argo CD

> **Argo CD** is a declarative, GitOps-based continuous delivery tool for Kubernetes that ensures the cluster state always matches the Git repository state.

#### Simple Meaning:

Git = Source of Truth
Kubernetes = Execution environment
Argo CD = Enforcer (keeps both in sync)

### 1.3 Traditional CI/CD Pipeline (Legacy Model)

#### Components:

* Jenkins / GitLab CI / GitHub Actions
* Shell scripts
* Python scripts
* Ansible playbooks
* Kubernetes kubectl commands

#### Flow:

```
Code → Build → Test → Security Scan → Docker Image → Deploy Script → Kubernetes
```

#### 🧨 Problem with Traditional CD:

Deployment logic is embedded in:

* Shell scripts
* Jenkins pipelines
* Manual steps

This leads to:

* Hard to maintain pipelines
* No real “desired state”
* No automatic drift detection

### ⚠️ 1.4 Major Drawback (Important Concept)

#### ❌ Problem: Configuration Drift

Example:

* DevOps deploys a config map
* Later a developer manually edits Kubernetes resources
* CI/CD pipeline is unaware

#### Result:

* Application behaves unpredictably after days
* No trace of who changed what

#### Real-World Scenario

Imagine:

* App deployed in QA works fine
* After 10 days, production fails
* No CI/CD pipeline ran

➤ Root cause:
Someone manually changed:

* ConfigMap
* Deployment YAML
* Environment variables

#### Analogy

Traditional CI/CD is like:

> “A waiter serving food, but customers can randomly change ingredients after delivery.”

GitOps is like:

> “A smart waiter who keeps checking the recipe book and replaces any incorrect dish automatically.”

### 1.5 Why Kubernetes Multi-Cluster Exists

Organizations don’t use one cluster.

They use multiple clusters for:

#### Environments:

* Development
* QA
* Staging
* Production

#### Feature-based clusters:

* Feature-1 cluster
* Feature-2 cluster

#### Team-based clusters:

* Team A cluster
* Team B cluster

#### Example Application

We use a sample application:

> **Guestbook Application**

Scenario:

* Version 1 deployed across clusters
* Version 2 introduces fixes/security updates

Goal:

➤ Deploy V2 to multiple clusters simultaneously

### 1.6 Problem Statement

How do we:

* Deploy same application to multiple clusters
* Ensure consistency
* Prevent manual drift
* Centralize control

#### ❌ Traditional Approach

You would:

* Login to each cluster
* Run `kubectl apply`
* Or run scripts/Ansible

Problems:

* Not scalable
* Error-prone
* No audit tracking

#### ✅ GitOps Approach (Argo CD)

Instead:

* Git stores Kubernetes manifests
* Argo CD monitors Git continuously
* Any drift is automatically corrected

### 1.7 GitOps Definition

> GitOps is a methodology where Git is the single source of truth for infrastructure and application deployment.

#### Core Principles:

* Declarative system
* Version-controlled configuration
* Automated reconciliation

#### GitOps Flow

```
Git Repo → Argo CD → Kubernetes Cluster(s)
           ↑            ↓
      Continuous     Auto Sync
      Monitoring     Self Healing
```

#### Key Insight

Unlike CI/CD:

* CI ends at deployment
* GitOps continues after deployment

### 1.8 What We Are Building in This Project

We will create:

#### Infrastructure:

* 1 Hub cluster (Argo CD installed)
* 2 Spoke clusters (target clusters)

#### Architecture:

```bash
           ┌────────────────────┐
           │   Git Repository   │
           └─────────┬──────────┘
                     │           
                     ▼            
           ┌────────────────────┐
           │    Argo CD (Hub)   │
           └─────────┬──────────┘
         ┌───────────┴────────────┐
         ▼                        ▼
┌─────────────────┐     ┌─────────────────┐
│ Spoke Cluster 1 │     │ Spoke Cluster 2 │
└─────────────────┘     └─────────────────┘
```

### PROJECT PREVIEW

You will learn to:

* Create EKS clusters using eksctl
* Install Argo CD
* Expose Argo CD UI
* Add external clusters
* Deploy Guestbook app
* Enable auto-sync
* Demonstrate self-healing

### Interview Questions

#### Q1: Why do companies use multiple Kubernetes clusters?

**A:** To isolate environments (dev, QA, prod), improve security, scalability, and team independence.

#### Q2: What is the biggest drawback of traditional CI/CD?

**A:** Lack of state reconciliation and inability to detect manual changes in clusters.

#### Q3: What problem does GitOps solve?

**A:** It ensures cluster state always matches Git state and automatically fixes drift.

### Summary

* Traditional CI/CD uses scripts and manual deployment
* It suffers from drift and lack of traceability
* Kubernetes environments are multi-cluster by nature
* GitOps introduces Git as single source of truth
* Argo CD continuously reconciles desired vs actual state

## 2. Understanding GitOps & Why It Matters

### Objective

Understand:

* What GitOps really means
* Why Git is used as source of truth
* Why CD is separated from CI

### 2.1 GitOps Core Idea

Instead of:

```
CI Pipeline → kubectl apply → cluster
```

We use:

```
Git → Argo CD → Kubernetes
```

### 2.2 Separation of CI and CD

#### CI (Continuous Integration):

* Build code
* Run tests
* Create Docker images

#### CD (Continuous Delivery):

* Deploy manifests
* Ensure desired state
* Reconcile cluster drift

### 2.3 Why Git as Source of Truth?

Because Git provides:

* Version control
* Audit history
* Rollback capability
* Collaboration

### 2.4 Self-Healing Concept

If someone changes Kubernetes manually:

```
kubectl edit configmap
```

Argo CD:

* Detects mismatch
* Reverts change
* Restores Git state

#### Analogy

GitOps is like:

> “A thermostat that constantly checks room temperature and adjusts it back to desired level.”

### 2.5 Real Example

If Git says:

```yaml
replicas: 3
```

But cluster has:

```
replicas: 10
```

Argo CD:

* detects drift
* resets to 3

### 2.6 Benefits of GitOps

* Full automation
* Strong auditability
* Disaster recovery
* Multi-cluster consistency
* Reduced human errors

### Interview Questions

#### Q1: What is GitOps?

**A:** A methodology where Git is the single source of truth for infrastructure and deployment.

#### Q2: How does Argo CD ensure consistency?

**A:** By continuously comparing Git state with cluster state and reconciling differences.

#### Q3: Difference between CI/CD and GitOps?

**A:** CI/CD pushes changes; GitOps continuously pulls desired state.

## ➤ Next: Section 2

We will cover:

* Argo CD Deep Dive
* Architecture Breakdown
* Hub–Spoke Model Overview
* and Standalone Model Overview
