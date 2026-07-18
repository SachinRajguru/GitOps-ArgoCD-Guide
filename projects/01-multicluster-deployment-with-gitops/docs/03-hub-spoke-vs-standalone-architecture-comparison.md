
## 📄 GitOps Multi-Cluster Deployment with Argo CD (Hub–Spoke Model on AWS EKS)

## Section 3 — Deployment Models: Hub–Spoke vs Standalone

## 5. Hub–Spoke Model (Core Multi-Cluster Architecture)

### Learning Objectives

By the end of this section, you will understand:

* What Hub–Spoke architecture means in Kubernetes
* How Argo CD uses a central control plane
* Why enterprises prefer centralized GitOps control
* How multi-cluster deployment is orchestrated

### 5.1 What is Hub–Spoke Model?

> The Hub–Spoke model is a centralized architecture where one “Hub” cluster controls multiple “Spoke” clusters.

#### In Argo CD context:

* **Hub Cluster** → Runs Argo CD
* **Spoke Clusters** → Target Kubernetes clusters where apps are deployed

### 5.2 Architecture Diagram

```bash
                 ┌────────────────────┐
                 │   Git Repository   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │   HUB CLUSTER      │
                 │   (Argo CD)        │
                 └─────────┬──────────┘
          ┌────────────────┼─────────────────┐
          ▼                ▼                 ▼
 ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
 │ Spoke Cluster │ │ Spoke Cluster │ │ Spoke Cluster │
 │ Dev / QA      │ │ Feature 1     │ │ Production    │
 └───────────────┘ └───────────────┘ └───────────────┘
```

### 5.3 How It Works

1. Developers push Kubernetes manifests to Git
2. Argo CD in HUB cluster detects changes
3. Argo CD compares desired vs actual state
4. Changes are pushed to multiple spoke clusters
5. Spoke clusters automatically sync state

### 5.4 Key Idea

The instructor emphasizes:

> One centralized Argo CD instance can manage multiple Kubernetes clusters.

This is the **core value of Hub–Spoke model**.

### 5.5 Why Hub–Spoke Exists

#### Problems solved:

* Centralized control
* Easier upgrades
* Single pane of visibility
* Simplified governance
* Easier disaster recovery

#### Analogy

Hub–Spoke is like:

> Airport system

* Hub airport (Dubai / Doha) → controls flights
* Spoke airports → destinations

All traffic is coordinated centrally.

### 5.6 Advantages

* Central control plane
* Unified GitOps policy enforcement
* Easier monitoring
* Reduced operational overhead
* Better for enterprise DevOps teams

### ⚠️ 5.7 Disadvantages

* Single point of failure if not HA
* High resource load on hub cluster
* Complex scaling if too many clusters
* Requires strong RBAC/security

## 6. Standalone Model (Distributed Argo CD)

### 6.1 What is Standalone Model?

> Each Kubernetes cluster has its own independent Argo CD instance.

#### Architecture Diagram

```bash
 Git Repo
    │
    ▼
 ┌──────────────┐
 │ Argo CD #1   │ → Cluster A (Dev)
 ├──────────────┤
 │ Argo CD #2   │ → Cluster B (QA)
 ├──────────────┤
 │ Argo CD #3   │ → Cluster C (Prod)
 └──────────────┘
```

### 6.2 How It Works

* Each cluster runs its own Argo CD
* Each instance watches Git
* Each cluster syncs independently

#### Key Idea

> There is no central controller; every cluster is self-managed.

### 6.3 Advantages

* High fault isolation
* No single point of failure
* Easier scaling per cluster
* Independent upgrades

### ⚠️ 6.4 Disadvantages

* Hard to manage at scale
* Repeated configuration across clusters
* No centralized visibility
* Operational overhead increases

### 6.5 Hub–Spoke vs Standalone Comparison

| Feature        | Hub–Spoke        | Standalone    |
| -------------- | ---------------- | ------------- |
| Control        | Centralized      | Distributed   |
| Management     | Easy             | Hard at scale |
| Failure impact | High (if not HA) | Low           |
| Visibility     | Unified          | Fragmented    |
| Complexity     | Medium           | High          |
| Use case       | Enterprise       | Small teams   |

### 6.6 When to Use What?

#### Use Hub–Spoke when:

* Large organization
* Many clusters (10–100+)
* Central DevOps team
* Governance required

#### Use Standalone when:

* Small teams
* Few clusters
* Independent teams
* Minimal ops complexity

### 6.7 Key Insight (Important)

The instructor explains:

* Hub cluster = centralized Argo CD
* Spoke clusters = target clusters
* One Argo CD can manage multiple clusters

He also highlights:

* Real organizations use both models depending on scale

#### Analogy

Standalone model is like:

> Multiple independent kitchens, each cooking separately

Hub–Spoke is like:

> One master kitchen sending instructions to multiple branches

### Interview Questions

#### Q1: What is Hub–Spoke model in Argo CD?

**A:** A centralized architecture where one Argo CD instance manages multiple Kubernetes clusters.

#### Q2: What is Standalone model?

**A:** Each cluster runs its own Argo CD instance independently.

#### Q3: Which model is better for large enterprises?

**A:** Hub–Spoke model due to centralized management and governance.

#### Q4: What is the main risk of Hub–Spoke?

**A:** Single point of failure if not configured with high availability.

### Summary

* Hub–Spoke = centralized GitOps control plane
* Standalone = distributed independent control planes
* Hub–Spoke is best for enterprise-scale deployments
* Standalone is best for small isolated setups
* Tradeoff is between control vs independence

## ➤ Next: Section 4

We will now begin the **hands-on project implementation**, including:

* Creating Multiple AWS EKS Clusters Using `eksctl`
* Understanding the Cluster Creation Workflow
* CloudFormation Behind Amazon EKS
* Resolving Common Provisioning Errors (like VPC stack issues)
* Switching `kubectl` Contexts
