
## 📄 `Multi-Cluster GitOps Deployment using Argo CD (Hub–Spoke Model on AWS EKS)`

## Section 4 — Hands-on Lab: Creating Multiple AWS EKS Clusters

## 7. Creating Multiple AWS EKS Clusters (Hands-on Lab Foundation)

### Learning Objectives

By the end of this section, you will understand:

* How multiple Kubernetes clusters are created using `eksctl`
* Why we create Hub + Spoke clusters
* How AWS CloudFormation is involved under the hood
* Why cluster provisioning takes time (20–30 minutes)
* How naming conventions help in multi-cluster setups

### 7.1 Why We Need Multiple Clusters

Organizations typically have:

* Dev cluster
* QA cluster
* Pre-production cluster
* Production cluster
* Feature-specific clusters (Feature-1, Feature-2, etc.)

➤ This is why we simulate **multi-cluster environment** in this lab.

### 7.2 Lab Design

We will create:

* **Hub Cluster** → Central Argo CD cluster
* **Spoke Cluster 1** → Target workload cluster
* **Spoke Cluster 2** → Target workload cluster

### Architecture

```id="eksclusters1"
        ┌─────────────────────────┐
        │     HUB CLUSTER         │
        │   (Argo CD Installed)   │
        └────────────┬────────────┘
                     │
       ┌─────────────┴─────────────┐
       ▼                           ▼
┌───────────────┐           ┌───────────────┐
│ Spoke Cluster │           │ Spoke Cluster │
│      1        │           │      2        │
└───────────────┘           └───────────────┘
```

### 7.3 Tool Used — eksctl

> `eksctl` is a CLI tool for creating and managing AWS EKS clusters easily.

#### What it does:

* Automates Kubernetes cluster creation
* Uses AWS CloudFormation behind the scenes
* Handles VPC, node groups, IAM roles automatically

### 7.4 Important Insight

Cluster creation:

> “takes around 20–30 minutes”

#### Why?

Because AWS must provision:

* VPC
* Subnets
* Route tables
* EC2 worker nodes
* Security groups
* IAM roles

### 7.5 Naming Convention

We create:

* `hub-cluster`
* `spoke-cluster-1`
* `spoke-cluster-2`

#### Why naming matters:

* Helps identify role of each cluster
* Avoids confusion in kubeconfig
* Essential for multi-cluster management

#### Analogy

Think of clusters as:

* Hub = Head office
* Spoke 1 = Branch office A
* Spoke 2 = Branch office B

### ⚠️ 7.6 Real-World Practice

In enterprises:

* Each cluster may belong to a different AWS account
* Or different regions (us-east-1, us-west-2)
* Or different business units

## 8. AWS EKS Cluster Creation Under the Hood

### 8.1 What Happens When You Run eksctl?

When you execute:

```bash
eksctl create cluster
```

AWS internally triggers:

#### CloudFormation Stack Creation

This includes:

* VPC stack
* EKS control plane
* Node group stack

### 8.2 CloudFormation Role (Important Concept)

> EKS cluster creation is backed by CloudFormation stacks.

If something fails:

* It appears in CloudFormation console
* Often due to VPC conflicts or leftover resources

### ❌ 8.3 Common Error

#### Error:

> “CloudFormation stack already exists”

#### Meaning:

* Previous cluster creation was incomplete
* Resources were not cleaned properly

### 8.4 Debugging Strategy (Very Important)

Instead of guessing:

* Check CloudFormation console
* Check VPC dashboard
* Identify stuck stacks
* Delete conflicting resources

#### Real-World Insight

> DevOps engineers spend significant time debugging infrastructure provisioning issues, not just writing code.

### 8.5 Fix Process

Steps:

1. Go to AWS CloudFormation
2. Find failed stack
3. Identify stuck resources (like VPC)
4. Delete stack manually
5. Retry cluster creation

### 8.6 AWS Components Created

Each EKS cluster creates:

* VPC
* Subnets
* NAT Gateway
* EC2 instances
* Security Groups
* IAM roles

#### Analogy

Creating an EKS cluster is like:

> Building a complete city from scratch every time

### 8.7 Why We Run Clusters in Parallel

Because:

> “We executed `eksctl` in parallel for all clusters”

#### Benefit:

* Saves time
* Parallel provisioning of infrastructure

### Interview Questions

#### Q1: What does eksctl do?

**A:** It simplifies EKS cluster creation by automating AWS infrastructure provisioning using CloudFormation.

#### Q2: What AWS service is used internally by EKS?

**A:** AWS CloudFormation.

#### Q3: Why does cluster creation take 20–30 minutes?

**A:** Because AWS provisions networking, compute, IAM roles, and control plane components.

#### Q4: What is a CloudFormation stack error?

**A:** It indicates infrastructure creation failed due to existing or conflicting AWS resources.

### Summary

* We create 3 clusters: 1 hub + 2 spoke
* eksctl simplifies cluster provisioning
* AWS CloudFormation manages infrastructure
* Errors often come from leftover resources
* Debugging AWS infra is a core DevOps skill

## ➤ Next: Section 5

We will now cover:

* Switching Kubernetes Contexts
* kubeconfig Management
* Installing Argo CD on the Hub Cluster
* Exposing the Argo CD UI (NodePort vs Ingress)
* Understanding the Argo CD Namespace Structure
