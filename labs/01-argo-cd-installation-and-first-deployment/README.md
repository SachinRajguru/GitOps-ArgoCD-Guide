
## 📄 `Lab 01: Argo CD Installation and First Deployment`

## Table of Contents

- [1. What This Lab Is About](#1-what-this-lab-is-about)
- [2. What We Are Building](#2-what-we-are-building)
- [3. Lab Architecture](#3-lab-architecture)
- [4. Important Concepts Before Starting](#4-important-concepts-before-starting)
  - [GitOps](#gitops)
  - [Argo CD](#argo-cd)
  - [Desired State](#desired-state)
  - [Actual State](#actual-state)
- [5. Prerequisites](#5-prerequisites)
  - [System Requirements](#system-requirements)
- [6. Kubernetes Environment Options](#6-kubernetes-environment-options)
- [7. Tool Installation](#7-tool-installation)
  - [7.1 Install kubectl](#71-install-kubectl)
    - [Windows](#windows)
    - [Linux](#linux)
    - [macOS](#macos)
  - [7.2 Install Minikube](#72-install-minikube)
  - [7.3 Install Argo CD CLI](#73-install-argo-cd-cli)
- [8. Start Kubernetes Cluster](#8-start-kubernetes-cluster)
- [9. Install Argo CD](#9-install-argo-cd)
  - [Verify Installation](#verify-installation)
- [10. Access Argo CD Dashboard](#10-access-argo-cd-dashboard)
- [11. Login Credentials](#11-login-credentials)
- [12. Deploy First Application](#12-deploy-first-application)
- [13. Lab Document Sequence](#13-lab-document-sequence)
  - [01-installation-and-core-components.md](#01-installation-and-core-componentsmd)
  - [02-ui-access-authentication-and-first-deployment.md](#02-ui-access-authentication-and-first-deploymentmd)
  - [03-reconciliation-sync-and-repository-structures.md](#03-reconciliation-sync-and-repository-structuresmd)
  - [04-cli-operations-and-application-management.md](#04-cli-operations-and-application-managementmd)
- [14. Common Issues](#14-common-issues)
- [15. What We Will Learn](#15-what-we-will-learn)
- [Final Outcome](#final-outcome)

## 1. What This Lab Is About

In this lab, we will install and configure **Argo CD** on a Kubernetes cluster and deploy our first application using the GitOps approach.

The goal is not only to install Argo CD but to understand:

* How Argo CD works internally
* How Kubernetes resources are managed through Git
* How applications move from Git → Argo CD → Kubernetes
* How automated synchronization works

By the end of this lab, we will have our first working GitOps deployment.

## 2. What We Are Building

Traditional Kubernetes deployment looks like:

```
Developer
    |
    |
kubectl apply
    |
    |
Kubernetes Cluster
```

The developer directly pushes changes into Kubernetes.

With GitOps:

```
Developer
    |
    |
Git Repository
(Source of Truth)
    |
    |
Argo CD
    |
    |
Kubernetes Cluster
```

Git becomes the desired state.

Argo CD continuously watches Git and makes Kubernetes match that state.

## 3. Lab Architecture

```
                   Developer
                       │
             Pushes Code & Manifests
                       │
                       ▼
                 Git Repository
                (Source of Truth)
                       │
                       ▼
                 Argo CD Server
            (Continuously Watches Git)
                       │
                       ▼
             Kubernetes API Server
                       │
                       ▼
             Kubernetes Cluster
          ┌────────────┼────────────┐
          ▼            ▼            ▼
     Deployment     Service     ConfigMap
          │
          ▼
      ReplicaSet
          │
          ▼
         Pods
```

## 4. Important Concepts Before Starting

Before running commands, understand these terms.

### GitOps

GitOps is a deployment methodology where:

* Git stores the desired Kubernetes configuration
* Git acts as the single source of truth
* A GitOps controller continuously compares Git state with cluster state
* Changes are automatically synchronized into Kubernetes

Example:

Git says:

```yaml
replicas: 3
```

Kubernetes currently has:

```
replicas: 1
```

Argo CD detects the difference and fixes it.

### Argo CD

Argo CD is a Kubernetes-native continuous delivery tool.

Its responsibility:

```
Monitor Git Repository
          |
          ▼
Compare Desired State with Cluster State
          |
          ▼
Detect Differences (Drift)
          |
          ▼
Synchronize Kubernetes Resources
          |
          ▼
Maintain Desired State
```

### Desired State

The configuration we want.

Example:

```yaml
replicas: 3
image: nginx:latest
```

Stored inside Git.

### Actual State

What is currently running inside Kubernetes.

Example:

```
Deployment:
replicas: 2
```

Argo CD continuously compares:

```
Desired State != Actual State

        ↓

Sync Required
```

## 5. Prerequisites

Before starting, make sure we have:

### System Requirements

Recommended:

| Resource | Requirement                 |
| -------- | --------------------------- |
| OS       | Linux / macOS / Windows WSL |
| RAM      | Minimum 8GB                 |
| CPU      | 2+ cores                    |
| Disk     | 10GB free                   |
| Internet | Required                    |

## 6. Kubernetes Environment Options

This lab can run on:

### Option 1: Local Kubernetes (Recommended for Learning)

Examples:

* Minikube
* Docker Desktop Kubernetes
* Kind

Architecture:

```
Laptop
 |
 |
Minikube
 |
 |
Argo CD
```

### Option 2: Cloud Kubernetes

Examples:

* AWS EKS
* Azure AKS
* Google GKE

Architecture:

```
Cloud Kubernetes Cluster

        |
        |
      Argo CD
```

For learning purposes, we will use:

```
Minikube Kubernetes Cluster
```

because:

* Free
* Runs locally
* Easy cleanup
* Perfect for understanding Argo CD

## 7. Tool Installation

Install tools in this order:

```
1. kubectl
2. Minikube
3. Argo CD CLI
4. Helm (optional)
```

### Windows / Linux / macOS Tool Setup

### 7.1 Install kubectl

#### Windows

PowerShell:

```powershell
choco install kubernetes-cli -y
```

Verify:

```powershell
kubectl version --client
```

#### Linux

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/
```

Verify:

```bash
kubectl version --client
```

#### macOS

```bash
brew install kubectl
```

Verify:

```bash
kubectl version --client
```

### 7.2 Install Minikube

#### Windows

```powershell
choco install minikube
```

#### Linux

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

#### macOS

```bash
brew install minikube
```

Verify:

```bash
minikube version
```

### 7.3 Install Argo CD CLI

#### Linux

```bash
curl -sSL -o argocd \
https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

chmod +x argocd

sudo mv argocd /usr/local/bin/
```

#### macOS

```bash
brew install argocd
```

#### Windows

```powershell
choco install argocd-cli
```

Verify:

```bash
argocd version
```

## 8. Start Kubernetes Cluster

Start Minikube:

```bash
minikube start
```

### What happens?

Minikube creates:

* Kubernetes control plane
* Worker node
* Container runtime
* Networking

Verify:

```bash
kubectl get nodes
```

Expected:

```
NAME       STATUS
minikube   Ready
```

## 9. Install Argo CD

Create namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD:

```bash
kubectl apply \
-n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Verify Installation

```bash
kubectl get pods -n argocd
```

Expected:

```
argocd-server       Running
argocd-controller   Running
argocd-repo-server Running
```

## 10. Access Argo CD Dashboard

Because Argo CD runs inside Kubernetes, it is not directly accessible.

Use:

```bash
kubectl port-forward svc/argocd-server \
-n argocd 8080:443
```

Open:

```
https://localhost:8080
```

## 11. Login Credentials

Get password:

```bash
kubectl get secret \
argocd-initial-admin-secret \
-n argocd \
-o jsonpath="{.data.password}" | base64 --decode
```

Login:

```
Username:
admin

Password:
<decoded password>
```

## 12. Deploy First Application

We will deploy:

```
Guestbook Application
```

Flow:

```
Git Repository

      ↓

Argo CD Application

      ↓

Kubernetes Deployment

      ↓

Running Pods
```

## 13. Lab Document Sequence

Follow these documents in order:

### 01-installation-and-core-components.md

Learn:

* Argo CD architecture
* Core components
* Namespace creation
* Installation process

### 02-ui-access-authentication-and-first-deployment.md

Learn:

* Access dashboard
* Login
* Create first application
* Deploy using UI

### 03-reconciliation-sync-and-repository-structures.md

Learn:

* Sync process
* Desired vs actual state
* Drift detection
* Repository structure

### 04-cli-operations-and-application-management.md

Learn:

* Argo CD CLI
* Application lifecycle
* Sync operations
* Troubleshooting

## ⚠️ 14. Common Issues

### ❌ kubectl cannot connect

Check:

```bash
kubectl cluster-info
```

Possible causes:

* Minikube not started
* Wrong context

Fix:

```bash
minikube start
```

### ❌ Argo CD pods not running

Check:

```bash
kubectl describe pod -n argocd
```

Possible causes:

* Low resources
* Image pull issue

### ❌ Cannot access UI

Check:

```bash
kubectl get svc -n argocd
```

Restart port forwarding:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## 15. What We Will Learn

After completing this lab:

* ✓ Kubernetes basics
* ✓ GitOps fundamentals
* ✓ Argo CD architecture
* ✓ Application deployment workflow
* ✓ Sync and reconciliation
* ✓ CLI-based management

## 🏁 Final Outcome

At the end of this lab, we will have:

```
Git Repository
       |
       |
    Argo CD
       |
       |
Kubernetes Cluster
       |
       |
Application Running
```

We have successfully completed our first GitOps deployment using Argo CD.
