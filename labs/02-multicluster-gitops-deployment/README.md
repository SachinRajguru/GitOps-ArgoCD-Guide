
## `GitOps Multi-Cluster Deployment using Argo CD (Hub–Spoke Model on AWS EKS)`

## Core Lab Sections

1. [Project Overview](#1-project-overview)
2. [Prerequisites & Local Environment Setup](#2-prerequisites--local-environment-setup)
3. [Provision Amazon EKS Clusters (Hub–Spoke Infrastructure Setup)](#3-provision-amazon-eks-clusters-hubspoke-infrastructure-setup)
4. [Install Argo CD on the Hub Cluster](#4-install-argo-cd-on-the-hub-cluster)
5. [Expose Argo CD UI (Lab Access Setup)](#5-expose-argo-cd-ui-lab-access-setup)
6. [Login to Argo CD & Security Configuration](#6-login-to-argo-cd--security-configuration)
7. [Connect Spoke Clusters to Argo CD (Multi-Cluster Registration)](#7-connect-spoke-clusters-to-argo-cd-multi-cluster-registration)
8. [Deploy Sample Application (Guestbook) using GitOps](#8-deploy-sample-application-guestbook-using-gitops)
9. [Drift Detection & Self-Healing (GitOps Validation)](#9-drift-detection--self-healing-gitops-validation)
10. [Troubleshooting & Common Issues (Real-World Debug Guide)](#10-troubleshooting--common-issues-real-world-debug-guide)
11. [Cluster Cleanup & Cost Management (AWS Teardown Guide)](#11-cluster-cleanup--cost-management-aws-teardown-guide)
12. [Real-World Use Cases & Industry Applications](#12-real-world-use-cases--industry-applications)

## 1. Project Overview

### Objective

This project demonstrates how to build a **GitOps-driven multi-cluster Kubernetes deployment platform** on **Amazon EKS** using **Argo CD**.

Instead of manually deploying applications, the deployment process is fully automated through Git, where Argo CD continuously reconciles the live cluster state with the desired state stored in the repository.

> **Git → Argo CD → Multiple Kubernetes Clusters → Continuous Reconciliation**

Upon detecting any configuration drift, Argo CD automatically restores the cluster to match the desired configuration defined in Git.

### Learning Objectives

After completing this lab, you will be able to:

* Provision multiple Amazon EKS clusters using `eksctl`
* Configure a Hub–Spoke Kubernetes architecture
* Install and configure Argo CD as the GitOps control plane
* Register multiple Kubernetes clusters with Argo CD
* Deploy applications using GitOps workflows
* Enable automatic synchronization, pruning, and self-healing
* Validate drift detection and continuous reconciliation
* Perform complete infrastructure cleanup to avoid unnecessary AWS charges

### Project Architecture

You will build the following production-style environment:

* **1 Hub Cluster** – Hosts the Argo CD control plane
* **2 Spoke Clusters** – Separate Development and Production environments
* **GitHub Repository** – Single source of truth for Kubernetes manifests
* **Automated GitOps Pipeline** – Continuous deployment and reconciliation across multiple clusters

### Technology Stack

| Component         | Purpose                                  |
| ----------------- | ---------------------------------------- |
| AWS               | Cloud infrastructure provider            |
| Amazon EKS        | Managed Kubernetes clusters              |
| Kubernetes        | Container orchestration platform         |
| Argo CD           | GitOps continuous delivery controller    |
| GitHub            | Source of truth for Kubernetes manifests |
| eksctl            | Provision and manage Amazon EKS clusters |
| kubectl           | Kubernetes command-line interface        |
| Helm *(Optional)* | Kubernetes package manager               |

### Project Workflow

```
Developer
     │
     ▼
Git Commit
     │
     ▼
GitHub Repository
     │
     ▼
Argo CD (Hub Cluster)
     │
     ▼
Detect Repository Changes
     │
     ▼
Compare Desired State vs Live State
     │
     ▼
Deploy Changes
     │
     ▼
Dev Cluster           Prod Cluster
     │                     │
     └──────────┬──────────┘
                ▼
      Continuous Reconciliation
```

### Hub–Spoke Architecture

```
                ┌─────────────────────┐
                │     GitHub Repo     │
                │  (Source of Truth)  │
                └─────────┬───────────┘
                          │
                          ▼
                ┌─────────────────────┐
                │     HUB CLUSTER     │
                │  (Argo CD Control)  │
                └─────────┬───────────┘
                          │
          ┌────────────────────────────────┐
          ▼                                ▼
      Dev Cluster                     Prod Cluster
    (Spoke Cluster)                  (Spoke Cluster)
```

### GitOps Principles Applied

This project follows the core GitOps model:

* Git acts as the **single source of truth**
* Argo CD continuously monitors the Git repository
* Kubernetes maintains the desired application state
* Configuration drift is automatically detected and corrected
* Application delivery remains consistent across all registered clusters

### Expected Outcome

By the end of this lab, you will have implemented:

* Multi-cluster Kubernetes architecture
* Centralized GitOps control plane
* Automated application deployment
* Continuous synchronization between Git and Kubernetes
* Drift detection with self-healing
* Production-style Hub–Spoke deployment architecture on Amazon EKS

### Skills Demonstrated

This project showcases practical experience with:

* Kubernetes Administration
* Amazon EKS
* GitOps
* Argo CD
* Multi-Cluster Management
* Infrastructure Provisioning
* Continuous Deployment (CD)
* Platform Engineering
* Cloud-Native Operations
* Drift Detection & Self-Healing

## 2. Prerequisites & Local Environment Setup

### Objective

Prepare your local machine and AWS environment for provisioning and managing multiple Amazon EKS clusters.

### 2.1 System Requirements

Ensure your system meets the following minimum requirements.

| Component        | Requirement                                             |
| ---------------- | ------------------------------------------------------- |
| Operating System | Linux, macOS, or Windows (WSL2 recommended for Windows) |
| RAM              | 8 GB minimum (16 GB recommended)                        |
| CPU              | 4 Cores or higher                                       |
| AWS Account      | AWS Free Tier (or equivalent)                           |

### 2.2 AWS Requirements

Before proceeding, ensure you have:

* AWS Account
* IAM User with permissions to create and manage EKS resources
* AWS Access Key ID
* AWS Secret Access Key
* Internet connectivity

### 2.3 Required IAM Permissions

The IAM user should have the following permissions:

| IAM Policy               | Purpose                                          |
| ------------------------ | ------------------------------------------------ |
| AmazonEKSFullAccess      | Create and manage Amazon EKS clusters            |
| IAMFullAccess            | Create and manage IAM roles required by EKS      |
| EC2FullAccess            | Provision worker nodes and networking resources  |
| CloudFormationFullAccess | Deploy AWS infrastructure through CloudFormation |

> **Production Note:** Follow the Principle of Least Privilege (PoLP) by granting only the minimum required permissions instead of full administrative access.

### 2.4 Install Required CLI Tools

Install the following tools on your local machine.

| Tool              | Purpose                                    |
| ----------------- | ------------------------------------------ |
| AWS CLI           | Interact with AWS services                 |
| kubectl           | Manage Kubernetes clusters                 |
| eksctl            | Create and manage Amazon EKS clusters      |
| Helm *(Optional)* | Install and manage Kubernetes applications |

### 2.5 Install AWS CLI

AWS CLI enables communication with AWS services from the command line.

#### Windows

Option 1 (Recommended)

Download and install the AWS CLI MSI installer.

#### macOS

```bash
brew install awscli
```

Alternatively, install using the official AWS PKG installer.

#### Linux

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install
```

#### Verify Installation

```bash
aws --version
```

Expected Output

```text
aws-cli/2.x.x
```

### Configure AWS Credentials

```bash
aws configure
```

Provide the following values:

```text
AWS Access Key ID     : <ACCESS_KEY>
AWS Secret Access Key : <SECRET_KEY>
Default Region        : us-east-1
Default Output Format : json
```

### Validate AWS Authentication

```bash
aws sts get-caller-identity
```

Expected Result

* AWS account information is returned.
* Successful output confirms that the CLI is authenticated and authorized.

### 2.6 Install kubectl

`kubectl` is the Kubernetes command-line tool used to interact with Kubernetes clusters.

#### Windows

Option 1

```powershell
choco install kubernetes-cli
```

Option 2

```powershell
winget install Kubernetes.kubectl
```

#### macOS

```bash
brew install kubectl
```

#### Linux

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/
```

#### Verify Installation

```bash
kubectl version --client
```

Expected Result

The client version of `kubectl` is displayed successfully.

### 2.7 Install eksctl

`eksctl` is the official CLI for creating and managing Amazon EKS clusters.

#### Windows

Option 1

```powershell
choco install eksctl
```

Option 2

```powershell
winget install Weaveworks.eksctl
```

#### macOS

```bash
brew tap weaveworks/tap

brew install weaveworks/tap/eksctl
```

#### Linux

```bash
curl --silent --location \
"https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
| tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```

#### Verify Installation

```bash
eksctl version
```

Expected Output

```text
0.xx.x
```

### 2.8 Install Helm (Optional)

Helm simplifies Kubernetes application installation using charts.

> **Note:** Helm is optional for this lab because Argo CD is installed using Kubernetes manifests.

#### Windows

Option 1

```powershell
choco install kubernetes-helm
```

Option 2

```powershell
winget install Helm.Helm
```

#### macOS

```bash
brew install helm
```

#### Linux

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

#### Verify Installation

```bash
helm version
```

Expected Result

The installed Helm version is displayed successfully.

### 2.9 Verify Local Environment

Confirm that all required tools are installed and accessible.

```bash
aws --version

kubectl version --client

eksctl version

helm version
```

Expected Result

* AWS CLI is installed.
* `kubectl` is available.
* `eksctl` is installed.
* Helm (if installed) is accessible.

## Checkpoint

Before proceeding to the next section, verify that:

* AWS CLI is installed and authenticated.
* `kubectl` is installed and working.
* `eksctl` is installed.
* Helm is installed *(optional)*.
* AWS credentials are configured successfully.
* Your IAM user has the required permissions.

### Expected Outcome

At the end of this section, you will have:

* A fully configured local development environment.
* Authenticated access to your AWS account.
* All required CLI tools installed and verified.
* The system ready to provision Amazon EKS clusters in the next section.

## 3. Provision Amazon EKS Clusters (Hub–Spoke Infrastructure Setup)

### Objective

Provision three Amazon EKS clusters that form the foundation of the GitOps Hub–Spoke architecture.

* **Hub Cluster** – Hosts the Argo CD control plane.
* **Dev Cluster** – Target cluster for development workloads.
* **Prod Cluster** – Target cluster for production workloads.

### Cluster Topology

| Cluster        | Role  | Purpose                                           |
| -------------- | ----- | ------------------------------------------------- |
| `hub-cluster`  | Hub   | Hosts Argo CD and manages all registered clusters |
| `dev-cluster`  | Spoke | Development environment                           |
| `prod-cluster` | Spoke | Production environment                            |

### 3.1 What Happens During Cluster Creation?

Running the `eksctl create cluster` command automatically provisions the required AWS infrastructure, including:

* Amazon VPC
* Public and Private Subnets
* Security Groups
* IAM Roles
* Amazon EKS Control Plane
* Managed Worker Nodes (EC2)
* Kubernetes Networking
* Local `kubeconfig` Context

> **Note:** Cluster creation typically takes **10–15 minutes** per cluster.

### 3.2 Create the Hub Cluster

The Hub Cluster hosts the Argo CD control plane.

```bash
eksctl create cluster \
  --name hub-cluster \
  --region us-east-1 \
  --nodes 2 \
  --node-type t3.medium \
  --managed
```

#### Verify Cluster Creation

```bash
kubectl get nodes
```

Expected Output

```text
NAME                 STATUS   ROLES    AGE   VERSION
ip-xxx.xxx.xxx.xxx   Ready    <none>   xxm   v1.xx.x
ip-xxx.xxx.xxx.xxx   Ready    <none>   xxm   v1.xx.x
```

All worker nodes should display a **Ready** status.

### 3.3 Create the Development Cluster

```bash
eksctl create cluster \
  --name dev-cluster \
  --region us-east-1 \
  --nodes 2 \
  --node-type t3.medium \
  --managed
```

#### Verify

```bash
kubectl get nodes
```

Expected Result

Both worker nodes are in the **Ready** state.

### 3.4 Create the Production Cluster

```bash
eksctl create cluster \
  --name prod-cluster \
  --region us-east-1 \
  --nodes 2 \
  --node-type t3.medium \
  --managed
```

#### Verify

```bash
kubectl get nodes
```

Expected Result

Both worker nodes are in the **Ready** state.

### 3.5 Verify All Kubernetes Contexts

Each EKS cluster is automatically added to your local `kubeconfig`.

List all available contexts:

```bash
kubectl config get-contexts
```

Expected Output

```text
CURRENT   NAME
*         hub-cluster
          dev-cluster
          prod-cluster
```

> **Note:** Context names may include the full EKS ARN depending on your `kubeconfig`. Use the context name returned by your environment throughout the lab.

### 3.6 Switch Between Clusters

Use the appropriate Kubernetes context before performing cluster-specific operations.

#### Switch to Hub Cluster

```bash
kubectl config use-context hub-cluster
```

Verify

```bash
kubectl get nodes
```

#### Switch to Development Cluster

```bash
kubectl config use-context dev-cluster
```

Verify

```bash
kubectl get nodes
```

#### Switch to Production Cluster

```bash
kubectl config use-context prod-cluster
```

Verify

```bash
kubectl get nodes
```

### 3.7 Verify Running EKS Clusters

List all EKS clusters in the AWS account.

```bash
eksctl get cluster
```

Expected Output

```text
NAME           REGION
hub-cluster    us-east-1
dev-cluster    us-east-1
prod-cluster   us-east-1
```

## Checkpoint

Before proceeding, verify the following:

* Hub Cluster is created successfully.
* Development Cluster is created successfully.
* Production Cluster is created successfully.
* All worker nodes are in the **Ready** state.
* All Kubernetes contexts are available in `kubeconfig`.
* You can switch between clusters without errors.
* `eksctl get cluster` lists all three clusters.

### Common Issues

| Issue                        | Possible Cause                      | Resolution                                                           |
| ---------------------------- | ----------------------------------- | -------------------------------------------------------------------- |
| Cluster creation fails       | Insufficient IAM permissions        | Verify required IAM policies                                         |
| Worker nodes not ready       | Node provisioning still in progress | Wait a few minutes and check again                                   |
| Context not found            | `kubeconfig` not updated            | Run `aws eks update-kubeconfig` or recreate the cluster if necessary |
| Unable to connect to cluster | Incorrect AWS credentials or region | Verify `aws configure` settings                                      |

### Expected Outcome

At the end of this section, you will have:

* Three fully operational Amazon EKS clusters.
* A Hub–Spoke Kubernetes architecture.
* Kubernetes contexts configured for all clusters.
* Verified connectivity to each cluster.
* Infrastructure ready for installing Argo CD.

### Lab Progress

| Component                      | Status       |
| ------------------------------ | ------------ |
| Local Environment              | ✅ Completed  |
| Hub Cluster                    | ✅ Created    |
| Dev Cluster                    | ✅ Created    |
| Prod Cluster                   | ✅ Created    |
| Kubernetes Contexts            | ✅ Configured |
| Ready for Argo CD Installation | ✅ Yes        |

## 4. Install Argo CD on the Hub Cluster

### Objective

Install and configure **Argo CD** on the **Hub Cluster** to enable GitOps-based continuous delivery across multiple Kubernetes clusters.

Argo CD will act as the **central control plane** that:

* Monitors Git repositories
* Detects configuration drift
* Synchronizes Kubernetes state with Git
* Manages multiple clusters (Dev + Prod)

### 4.1 Switch to Hub Cluster

Ensure all Argo CD components are installed only on the Hub Cluster.

```bash id="hubctx1"
kubectl config use-context hub-cluster
```

Verify:

```bash id="hubctx2"
kubectl get nodes
```

Expected Output:

* 2 worker nodes in **Ready** state

### 4.2 Create Argo CD Namespace

Argo CD will be deployed in a dedicated namespace.

```bash id="ns1"
kubectl create namespace argocd
```

Verify:

```bash id="ns2"
kubectl get namespaces
```

Expected:

```text id="ns3"
argocd   Active
```

### 4.3 Install Argo CD

Deploy Argo CD using the official manifest.

```bash id="install1"
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### What This Installs

This command deploys the full Argo CD stack:

* API Server
* Repository Server
* Application Controller
* Redis
* Dex (authentication service)

### 4.4 Verify Argo CD Installation

Check all pods in the namespace.

```bash id="verify1"
kubectl get pods -n argocd
```

Expected Output (all should be Running):

```text id="verify2"
argocd-application-controller   Running
argocd-repo-server              Running
argocd-server                   Running
argocd-dex-server               Running
argocd-redis                    Running
```

### 4.5 Verify Services

```bash id="svc1"
kubectl get svc -n argocd
```

Expected Services:

* argocd-server
* argocd-repo-server
* argocd-application-controller
* argocd-redis

## Checkpoint

Before proceeding, ensure:

* Argo CD namespace exists
* All Argo CD pods are in **Running** state
* Services are created successfully

### Expected Outcome

At the end of this section:

* Argo CD is successfully installed on the Hub Cluster
* All core components are running
* The system is ready for external access configuration
* GitOps control plane is operational (internally)

### Key Concept

> Argo CD continuously watches Git repositories and ensures Kubernetes clusters always match the desired state defined in Git.

This enables:

* Automated deployments
* Drift detection
* Self-healing infrastructure
* Declarative cluster management

### Common Issues

| Issue                 | Cause                       | Fix                                      |
| --------------------- | --------------------------- | ---------------------------------------- |
| Pods stuck in Pending | Insufficient node resources | Wait for node readiness or scale cluster |
| Image pull errors     | Network restrictions        | Ensure internet access from nodes        |
| CrashLoopBackOff      | Temporary startup delay     | Wait 2–3 minutes and recheck             |
| Namespace not found   | Install step skipped        | Re-run namespace creation                |

### Lab Progress

| Component            | Status      |
| -------------------- | ----------- |
| EKS Clusters         | ✅ Completed |
| Hub Cluster          | ✅ Active    |
| Argo CD Namespace    | ✅ Created   |
| Argo CD Installation | ✅ Completed |
| Argo CD Pods         | ✅ Running   |
| UI Access            | ⏳ Next Step |

## 5. Expose Argo CD UI (Lab Access Setup)

### Objective

Expose the **Argo CD UI** from the Hub Cluster so it can be accessed from a browser for lab and learning purposes.

By default, Argo CD is only accessible inside the Kubernetes cluster. This section enables external access using a **NodePort service**.

> ⚠️ This approach is strictly for lab environments. Production systems should use an Ingress Controller with AWS ALB + HTTPS.

### 5.1 Identify Argo CD Server Service

Switch to Hub Cluster (if not already):

```bash id="ctx1"
kubectl config use-context hub-cluster
```

List services in Argo CD namespace:

```bash id="svc1"
kubectl get svc -n argocd
```

Locate:

* `argocd-server`

### 5.2 Modify Service Type to NodePort

Edit the service:

```bash id="edit1"
kubectl edit svc argocd-server -n argocd
```

Find:

```yaml
type: ClusterIP
```

Change to:

```yaml
type: NodePort
```

Save and exit.

### 5.3 Verify NodePort Assignment

Check updated service:

```bash id="svc2"
kubectl get svc -n argocd
```

Expected Output Example:

```text id="svc3"
argocd-server   NodePort   80:30812/TCP
```

* `30812` = NodePort (randomly assigned)
* This port will be used to access UI externally

### 5.4 Get Worker Node Public IP

Retrieve node details:

```bash id="node1"
kubectl get nodes -o wide
```

Identify:

* Public IP of any worker node

### 5.5 Access Argo CD UI

Open browser:

```text id="ui1"
http://<WORKER_NODE_PUBLIC_IP>:<NODE_PORT>
```

Example:

```text id="ui2"
http://3.120.45.22:30812
```

### 5.6 Expected Browser Behavior

When accessing for the first time, you may see:

* ⚠️ “Your connection is not private”
* ⚠️ SSL/TLS warning

#### Why this happens

* Argo CD uses **self-signed TLS certificates**
* Browser cannot verify certificate authority

#### How to proceed (Lab only)

* Click **Advanced**
* Click **Proceed (unsafe)**

### 5.7 Production Alternative (Important)

In real-world environments, NodePort should NOT be used.

Use instead:

* AWS Application Load Balancer (ALB)
* AWS ACM SSL Certificates
* Route53 Domain Mapping
* Kubernetes Ingress Controller

## Checkpoint

Before proceeding, verify:

* argocd-server service is in NodePort mode
* NodePort is assigned (e.g., 30000–32767 range)
* Worker node public IP is accessible
* UI opens in browser

### Expected Outcome

At the end of this section:

* Argo CD UI is accessible externally
* You can reach the login screen via browser
* Kubernetes control plane is now visually manageable
* System is ready for authentication setup

### Common Issues

| Issue                   | Cause                        | Fix                                |
| ----------------------- | ---------------------------- | ---------------------------------- |
| Site not reachable      | Security group blocking port | Allow NodePort range (30000–32767) |
| Connection timeout      | Wrong public IP              | Verify node external IP            |
| Service still ClusterIP | Edit not saved properly      | Re-run `kubectl edit svc`          |
| UI not loading          | Node not ready               | Check `kubectl get nodes`          |

### Lab Progress

| Component          | Status      |
| ------------------ | ----------- |
| EKS Clusters       | ✅ Done      |
| Argo CD Installed  | ✅ Done      |
| Argo CD UI Exposed | ✅ Done      |
| Browser Access     | ⏳ Next Step |
| Authentication     | ⏳ Pending   |

## 6. Login to Argo CD & Security Configuration

### Objective

Access the **Argo CD UI** and CLI by retrieving the **initial admin credentials**, completing authentication, and understanding basic security implications in a lab environment.

### 6.1 Retrieve Argo CD Admin Password

Argo CD creates a default **admin user** with a generated password stored in a Kubernetes secret.

```bash id="pwd1"
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

### 6.2 Decode the Password

Extract and decode the password:

```bash id="pwd2"
echo <BASE64_PASSWORD> | base64 --decode
```

Example:

```bash id="pwd3"
echo cGFzc3dvcmQ= | base64 --decode
```

Expected Output:

```text id="pwd4"
password
```

### 6.3 Login to Argo CD UI

Open browser:

```text id="ui1"
http://<WORKER_NODE_PUBLIC_IP>:<NODE_PORT>
```

#### Credentials

| Field    | Value                     |
| -------- | ------------------------- |
| Username | admin                     |
| Password | Decoded value from secret |

### 6.4 First-Time UI Warning

You may see:

* ⚠️ “Your connection is not private”
* ⚠️ SSL certificate warning

#### Why this happens

* Argo CD uses a **self-signed TLS certificate**
* No trusted Certificate Authority (CA) is attached

#### How to proceed (Lab only)

* Click **Advanced**
* Click **Proceed to site**

### 6.5 Login via Argo CD CLI (Optional but Recommended)

#### Login Command

```bash id="cli1"
argocd login <WORKER_NODE_PUBLIC_IP>:<NODE_PORT>
```

Example:

```bash id="cli2"
argocd login 3.120.45.22:30812
```

#### Enter Credentials

```text id="cli3"
Username: admin
Password: <decoded-password>
```

### 6.6 Verify Login

```bash id="cli4"
argocd version
```

Expected Output:

* Client version
* Server version

### 6.7 Understand Default Security Model

Argo CD default setup includes:

* Single admin user
* No RBAC configuration initially
* Self-signed TLS certificate
* Cluster-admin level permissions inside cluster

### 6.8 Security Notes (Important)

#### Lab Setup (Current State)

* NodePort exposure enabled
* Self-signed certificates used
* Default admin credentials active

#### Production Recommendations

In real-world environments:

* Use AWS ALB + HTTPS (ACM certificates)
* Configure DNS via Route53
* Enable RBAC roles and SSO (OIDC/SAML)
* Disable admin access or rotate credentials
* Use private cluster endpoints where possible

## Checkpoint

Before proceeding, verify:

* Admin password successfully retrieved
* Argo CD UI login is successful
* CLI login works (optional but recommended)
* You can see Argo CD dashboard

### Expected Outcome

At the end of this section:

* You are authenticated into Argo CD UI
* CLI access is configured (optional)
* You understand basic security limitations of lab setup
* Control plane is fully operational and ready for cluster registration

### Common Issues

| Issue               | Cause                       | Fix                              |
| ------------------- | --------------------------- | -------------------------------- |
| Secret not found    | Argo CD not fully installed | Wait and re-check pods           |
| Base64 decode error | Incorrect value copied      | Copy only `.data.password` field |
| Login fails         | Wrong IP or port            | Verify NodePort and node IP      |
| UI inaccessible     | Security group issue        | Allow inbound NodePort range     |

### Lab Progress

| Component            | Status      |
| -------------------- | ----------- |
| EKS Clusters         | ✅ Ready     |
| Argo CD Installed    | ✅ Done      |
| UI Access            | ✅ Done      |
| Authentication       | ✅ Completed |
| Cluster Registration | ⏳ Next Step |

## 7. Connect Spoke Clusters to Argo CD (Multi-Cluster Registration)

### Objective

Register **Dev** and **Prod (Spoke) clusters** with the **Argo CD Hub Cluster**, enabling centralized GitOps management across multiple Kubernetes environments.

After this step, Argo CD will be able to:

* Deploy applications to multiple clusters
* Monitor cluster states remotely
* Manage Dev and Prod environments independently
* Maintain Git as the single source of truth

### 7.1 Install Argo CD CLI

The Argo CD CLI is required to register external clusters and manage applications.

#### Windows

```powershell id="cli1"
choco install argocd-cli
```

Or manual install:

```powershell id="cli2"
Invoke-WebRequest `
  -Uri "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-windows-amd64.exe" `
  -OutFile "argocd.exe"
```

Move to system path:

```powershell id="cli3"
move argocd.exe C:\Windows\System32\
```

#### macOS

```bash id="cli4"
brew install argocd
```

#### Linux

```bash id="cli5"
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

chmod +x argocd

sudo mv argocd /usr/local/bin/
```

#### Verify Installation

```bash id="cli6"
argocd version
```

Expected Output:

* Client version displayed successfully

### 7.2 Login to Argo CD via CLI

```bash id="login1"
argocd login <ARGOCD_SERVER_IP>:<NODE_PORT>
```

Example:

```bash id="login2"
argocd login 3.120.45.22:30812
```

Enter credentials:

```text id="login3"
Username: admin
Password: <decoded-password>
```

### 7.3 Verify Kubernetes Contexts

Ensure all clusters are available locally.

```bash id="ctx1"
kubectl config get-contexts
```

Expected Output:

```text id="ctx2"
CURRENT   NAME
*         hub-cluster
          dev-cluster
          prod-cluster
```

### 7.4 Add Dev Cluster to Argo CD

```bash id="add1"
argocd cluster add dev-cluster
```

#### What happens internally?

Argo CD automatically:

* Creates a ServiceAccount in Dev cluster
* Assigns cluster-admin permissions
* Generates authentication token
* Stores cluster credentials securely in Argo CD

### 7.5 Add Prod Cluster to Argo CD

```bash id="add2"
argocd cluster add prod-cluster
```

### 7.6 Verify Registered Clusters

```bash id="list1"
argocd cluster list
```

Expected Output:

```text id="list2"
SERVER                          NAME        STATUS
https://kubernetes.default.svc in-cluster  Successful
https://...eks.amazonaws.com   dev-cluster Successful
https://...eks.amazonaws.com   prod-cluster Successful
```

### 7.7 Validate in Argo CD UI

Navigate to:

```
Settings → Clusters
```

You should see:

* in-cluster (Hub Cluster)
* dev-cluster
* prod-cluster

All must show **Connected / Successful**

### 7.8 Understand Cluster Registration Model

| Component           | Role                                 |
| ------------------- | ------------------------------------ |
| Hub Cluster         | Runs Argo CD control plane           |
| Spoke Clusters      | Target environments (Dev / Prod)     |
| ServiceAccount      | Auth mechanism created automatically |
| Cluster Credentials | Stored securely in Argo CD           |

## Checkpoint

Before proceeding, verify:

* Argo CD CLI is installed
* CLI login is successful
* All kubeconfig contexts exist
* Dev cluster is registered
* Prod cluster is registered
* Both clusters show “Successful” status in Argo CD

### Expected Outcome

At the end of this section:

* Argo CD has full control over multiple clusters
* Dev and Prod clusters are registered and accessible
* GitOps multi-cluster architecture is fully operational
* System is ready for application deployment

### Common Issues

| Issue               | Cause                  | Fix                                 |
| ------------------- | ---------------------- | ----------------------------------- |
| cluster add fails   | Wrong kube context     | Verify `kubectl config use-context` |
| permission denied   | IAM / RBAC mismatch    | Re-run `argocd cluster add`         |
| cluster not visible | CLI not logged in      | Re-login to Argo CD                 |
| context missing     | kubeconfig not updated | Recreate cluster or refresh config  |

### Lab Progress

| Component               | Status                  |
| ----------------------- | ----------------------- |
| EKS Clusters            | ✅ Done                  |
| Argo CD Installed       | ✅ Done                  |
| UI + Login              | ✅ Done                  |
| Dev Cluster Registered  | ✅ Done                  |
| Prod Cluster Registered | ✅ Done                  |
| Multi-Cluster GitOps    | 🚀 Ready for Deployment |

## 8. Deploy Sample Application (Guestbook) using GitOps

### Objective

Deploy a sample application (**Guestbook**) using **Argo CD GitOps workflow** into the **Dev Cluster**.

This section demonstrates:

* Git-based application deployment
* Kubernetes manifest management
* Argo CD application synchronization
* Multi-resource deployment (Deployment, Service, ConfigMap)

### 8.1 Application Structure (Git Repository)

Create the following directory structure in your GitHub repository:

```text id="repo1"
manifests/
└── guestbook/
    ├── deployment.yaml
    ├── service.yaml
    └── configmap.yaml
```

### 8.2 Kubernetes Manifests

#### Deployment

```yaml id="dep1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
spec:
  replicas: 2
  selector:
    matchLabels:
      app: guestbook
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
      - name: guestbook
        image: nginx:latest
        ports:
        - containerPort: 80
```

#### Service

```yaml id="svc1"
apiVersion: v1
kind: Service
metadata:
  name: guestbook-service
spec:
  type: NodePort
  selector:
    app: guestbook
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

#### ConfigMap

```yaml id="cfg1"
apiVersion: v1
kind: ConfigMap
metadata:
  name: guestbook-config
data:
  UI_MODE: "production"
```

### 8.3 Create Argo CD Application

#### Step 1: Open Argo CD UI

```text id="ui1"
http://<WORKER_NODE_PUBLIC_IP>:<NODE_PORT>
```

Login using admin credentials.


#### Step 2: Create New Application

Click:

```text id="ui2"
New Application
```

#### Step 3: Configure Application

| Field            | Value     |
| ---------------- | --------- |
| Application Name | guestbook |
| Project          | default   |
| Sync Policy      | Automatic |
| Auto-Prune       | Enabled   |
| Self Heal        | Enabled   |

#### Step 4: Source Configuration

| Field          | Value                                                 |
| -------------- | ----------------------------------------------------- |
| Repository URL | [https://github.com/](https://github.com/)<your-repo> |
| Revision       | main                                                  |
| Path           | manifests/guestbook                                   |

#### Step 5: Destination Configuration

| Field     | Value       |
| --------- | ----------- |
| Cluster   | dev-cluster |
| Namespace | default     |

#### Step 6: Create Application

Click:

```text id="ui3"
Create
```

### 8.4 Sync Behavior

Once created, Argo CD will:

* Pull manifests from GitHub
* Compare desired state vs live state
* Automatically deploy resources to Dev Cluster

### 8.5 Enable GitOps Features

Ensure the following are enabled:

* Auto Sync
* Self Heal
* Prune Resources

#### Meaning of Each Feature

| Feature   | Behavior                          |
| --------- | --------------------------------- |
| Auto Sync | Automatically applies Git changes |
| Self Heal | Reverts manual cluster changes    |
| Prune     | Deletes removed Git resources     |

### 8.6 Verify Deployment

#### Check Argo CD UI

Application status should show:

```text id="status1"
Synced ✔
Healthy ✔
```

#### Check via kubectl

Switch to Dev Cluster:

```bash id="kubectx1"
kubectl config use-context dev-cluster
```

Verify pods:

```bash id="kubectl1"
kubectl get pods
```

Expected:

* 2 running pods for guestbook

Verify service:

```bash id="kubectl2"
kubectl get svc
```

Expected:

* guestbook-service exposed via NodePort

### 8.7 Access Application

Open in browser:

```text id="app1"
http://<NODE_IP>:30080
```

Expected:

* Nginx default page (Guestbook frontend placeholder)

## Checkpoint

Before proceeding, verify:

* GitHub repo contains manifests
* Argo CD application created successfully
* Application is in Synced state
* Pods are running in Dev cluster
* Service is exposed via NodePort
* Application is accessible in browser

### Expected Outcome

At the end of this section:

* GitOps pipeline is fully functional
* Application is deployed using Argo CD
* Dev cluster is actively managed via Git
* Kubernetes resources are synchronized automatically
* First end-to-end GitOps workflow is complete

### Common Issues

| Issue              | Cause                             | Fix                          |
| ------------------ | --------------------------------- | ---------------------------- |
| OutOfSync status   | Git not synced                    | Click Sync manually          |
| Pods not running   | Image pull delay                  | Wait and recheck             |
| App not accessible | Security group missing port 30080 | Allow NodePort range         |
| Wrong repo path    | Incorrect Git structure           | Verify `manifests/guestbook` |
| Sync fails         | YAML error                        | Validate manifest syntax     |

### Lab Progress

| Component                  | Status       |
| -------------------------- | ------------ |
| EKS Clusters               | ✅ Done       |
| Argo CD Setup              | ✅ Done       |
| Multi-Cluster Registration | ✅ Done       |
| Guestbook Deployment       | 🚀 Completed |
| GitOps Pipeline            | 🚀 Active    |
| Drift Detection            | ⏳ Next Step  |

## 9. Drift Detection & Self-Healing (GitOps Validation)

### Objective

Validate the core power of **GitOps with Argo CD** by introducing a manual change in the Kubernetes cluster and observing:

* Drift detection (OutOfSync state)
* Automatic reconciliation
* Self-healing behavior

This proves that **Git is the single source of truth** and cluster state cannot permanently diverge.

### 9.1 Switch to Dev Cluster

All tests will be performed on the Dev Cluster.

```bash id="ctx1"
kubectl config use-context dev-cluster
```

Verify:

```bash id="ctx2"
kubectl get nodes
```

### 9.2 Identify ConfigMap

List ConfigMaps:

```bash id="cm1"
kubectl get configmap
```

Expected:

* `guestbook-config`

### 9.3 Modify Live Cluster (Manual Change)

Edit the ConfigMap directly in the cluster:

```bash id="cm2"
kubectl edit configmap guestbook-config
```

#### Change This Value

Before (Git source of truth):

```yaml id="old"
UI_MODE: "production"
```

Change to:

```yaml id="new"
UI_MODE: "test"
```

Save and exit.

### 9.4 Observe Drift Detection in Argo CD

Go to Argo CD UI:

```text id="ui1"
guestbook application → Status
```

#### Expected Behavior

Argo CD detects mismatch:

```text id="status1"
OutOfSync ❌
```

This means:

* Live cluster state ≠ Git desired state
* Configuration drift is detected

### 9.5 Automatic Self-Healing

If **Auto Sync + Self Heal** is enabled, Argo CD will automatically:

1. Detect the drift
2. Compare with Git repository
3. Reapply correct configuration
4. Restore original state

#### Final State After Reconciliation

```yaml id="final"
UI_MODE: "production"
```

Argo CD restores the value from Git.

### 9.6 Verify Recovery via CLI

```bash id="verify1"
kubectl get configmap guestbook-config -o yaml
```

Expected:

* Value is reverted to `production`

### 9.7 Key Observation

| Action                | Result           |
| --------------------- | ---------------- |
| Manual cluster change | Drift introduced |
| Argo CD detection     | OutOfSync state  |
| Self-heal enabled     | Auto correction  |
| Final state           | Matches Git      |

### 9.8 GitOps Principle Demonstrated

> Any manual change in Kubernetes is temporary unless reflected in Git.

#### Core Behavior:

* Git = Desired State
* Kubernetes = Runtime State
* Argo CD = Reconciliation Engine

## Checkpoint

Before proceeding, verify:

* Manual ConfigMap edit completed
* Argo CD shows OutOfSync (initially)
* Auto Sync is enabled
* Value is reverted to Git state
* Cluster state matches repository

### Expected Outcome

At the end of this section:

* Drift detection is successfully demonstrated
* Self-healing behavior is validated
* GitOps reconciliation loop is understood
* Cluster enforces Git-defined state automatically
* Manual changes are overridden by Argo CD

### Common Issues

| Issue                  | Cause                            | Fix                             |
| ---------------------- | -------------------------------- | ------------------------------- |
| No OutOfSync status    | Auto-sync immediately reconciles | Observe logs/UI carefully       |
| Value not reverting    | Self-heal disabled               | Enable Auto Sync + Self Heal    |
| Wrong ConfigMap edited | Wrong resource selected          | Confirm name `guestbook-config` |
| UI not updating        | Browser cache delay              | Refresh Argo CD UI              |

### Lab Progress

| Component              | Status       |
| ---------------------- | ------------ |
| EKS Clusters           | ✅ Done       |
| Argo CD Installed      | ✅ Done       |
| Multi-Cluster Setup    | ✅ Done       |
| Application Deployment | ✅ Done       |
| GitOps Workflow        | ✅ Active     |
| Drift Detection        | 🚀 Completed |
| Self-Healing           | 🚀 Verified  |
| Troubleshooting        | ⏳ Next Step  |

## 10. Troubleshooting & Common Issues (Real-World Debug Guide)

### Objective

This section provides a **practical debugging playbook** for issues commonly encountered in:

* Amazon EKS clusters
* Argo CD installations
* GitOps deployments
* NodePort exposure
* IAM and networking failures

> This is a **production-style troubleshooting guide**, not theory.

### 10.1 Kubernetes Cluster Issues

### Issue: Nodes Not Ready

#### Symptom

```text id="t1"
kubectl get nodes → NotReady
```

#### Causes

* Node provisioning still in progress
* Insufficient EC2 capacity
* IAM role issues

#### Fix

```bash id="t2"
kubectl get nodes --watch
```

Wait 5–10 minutes.

### Issue: Cluster Context Not Found

#### Symptom

```text id="t3"
error: context not found
```

#### Fix

```bash id="t4"
kubectl config get-contexts
```

Switch context:

```bash id="t5"
kubectl config use-context dev-cluster
```

### 10.2 Argo CD Installation Issues

### Issue: Pods Stuck in Pending

#### Symptom

```text id="t6"
argocd-server Pending
```

#### Causes

* Insufficient CPU/Memory
* Node not ready

#### Fix

```bash id="t7"
kubectl describe pod <pod-name> -n argocd
```

Check events and scale cluster if needed.

### Issue: CrashLoopBackOff

#### Fix Steps

```bash id="t8"
kubectl logs -n argocd deployment/argocd-server
```

* Wait for services to stabilize
* Restart deployment if needed

### 10.3 Argo CD UI Access Issues

### Issue: Site Not Reachable

#### Causes

* Security Group not configured
* NodePort blocked

#### Fix

Ensure inbound rule exists:

| Type | Port        | Source  |
| ---- | ----------- | ------- |
| TCP  | 30000–32767 | Your IP |

Then retry:

```text id="t9"
http://<NODE_IP>:<NODE_PORT>
```

### Issue: Wrong Node IP

#### Fix

```bash id="t10"
kubectl get nodes -o wide
```

Use **EXTERNAL-IP**, not internal IP.

### 10.4 Argo CD Login Issues

### Issue: Invalid Credentials

#### Fix

Re-fetch password:

```bash id="t11"
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

Decode again:

```bash id="t12"
echo <base64> | base64 --decode
```

### Issue: CLI Login Fails

#### Fix

```bash id="t13"
argocd login <IP>:<NODEPORT>
```

Ensure:

* Correct IP
* Correct port
* UI is accessible

### 10.5 Cluster Registration Issues

### Issue: argocd cluster add Fails

#### Cause

* Wrong kube context
* Missing permissions

#### Fix

```bash id="t14"
kubectl config get-contexts
```

Then retry:

```bash id="t15"
argocd cluster add dev-cluster
```

### Issue: Cluster Not Visible in Argo CD

#### Fix

Check:

```bash id="t16"
argocd cluster list
```

If missing:

* Re-run cluster add command
* Ensure CLI is logged in

### 10.6 GitOps Deployment Issues

### Issue: OutOfSync (Not Expected)

#### Causes

* Git repo path incorrect
* YAML syntax error
* Missing manifests

#### Fix

Verify structure:

```text id="t17"
manifests/guestbook/
```

Check Argo CD application path.

### Issue: Application Not Creating Pods

#### Fix

```bash id="t18"
kubectl get events
kubectl describe pod <pod>
```

Common cause:

* Image pull delay (nginx:latest)

### 10.7 Service & NodePort Issues

### Issue: Application Not Accessible

#### Causes

* Security group missing port
* Wrong NodePort
* DNS/IP mismatch

#### Fix

Check service:

```bash id="t19"
kubectl get svc
```

Ensure:

* NodePort exists (30000–32767)
* Correct cluster selected

### 10.8 IAM & AWS Issues

### Issue: Access Denied

#### Fix

Verify IAM:

* AmazonEKSFullAccess
* EC2FullAccess
* IAMFullAccess

Check identity:

```bash id="t20"
aws sts get-caller-identity
```

### 10.9 Git Sync Issues

### Issue: Changes Not Reflecting

#### Fix

* Verify GitHub repo URL
* Confirm branch = main
* Confirm correct path

Then force sync:

```text id="t21"
Argo CD UI → Sync → Force
```

### 10.10 Debugging Toolkit (Must Know Commands)

```bash id="t22"
kubectl get pods -A
kubectl get nodes -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get svc
kubectl get events
kubectl config get-contexts
argocd cluster list
argocd app list
```

## Checkpoint

Before proceeding, ensure:

* You can debug node issues
* You can fix UI access problems
* You can resolve Argo CD login issues
* You can troubleshoot Git sync failures
* You can identify NodePort issues
* You understand IAM-related errors

### Expected Outcome

At the end of this section:

* You can independently debug EKS clusters
* You can resolve Argo CD deployment issues
* You can fix GitOps synchronization problems
* You can troubleshoot real production-like failures
* You have a complete DevOps debugging toolkit

### Lab Progress

| Component         | Status       |
| ----------------- | ------------ |
| EKS Clusters      | ✅ Done       |
| Argo CD Setup     | ✅ Done       |
| GitOps Deployment | ✅ Done       |
| Drift Detection   | ✅ Done       |
| Troubleshooting   | 🚀 Completed |
| Cleanup           | ⏳ Next Step  |

## 11. Cluster Cleanup & Cost Management (AWS Teardown Guide)

### Objective

Safely delete all AWS resources created in this lab to:

* Prevent unnecessary AWS billing
* Clean up EKS infrastructure
* Remove Kubernetes clusters in correct order
* Ensure complete environment teardown

> ⚠️ Important: Always perform cleanup after completing the lab unless you explicitly want to keep resources running.

### 11.1 Why Cleanup is Important

Amazon EKS resources incur costs for:

* EC2 worker nodes
* Load balancers
* Networking components (VPC, NAT gateways)
* Control plane (EKS cluster)

Leaving clusters running may lead to **unexpected AWS charges**.

### 11.2 List All Active Clusters

```bash id="c1"
eksctl get cluster
```

Expected Output:

```text id="c2"
NAME           REGION
hub-cluster    us-east-1
dev-cluster    us-east-1
prod-cluster   us-east-1
```

### 11.3 Switch to Safe Context

Before deletion, switch to a cluster context that still exists:

```bash id="c3"
kubectl config use-context hub-cluster
```

### 11.4 Delete Dev Cluster

```bash id="c4"
eksctl delete cluster \
  --name dev-cluster \
  --region us-east-1
```

#### Expected Behavior

* Worker nodes terminated
* VPC cleanup initiated
* Cluster removed from AWS console

⏳ This process may take 10–15 minutes

### 11.5 Delete Prod Cluster

```bash id="c5"
eksctl delete cluster \
  --name prod-cluster \
  --region us-east-1
```

#### Expected Behavior

* All workloads removed
* Node groups deleted
* Networking resources cleaned up

### 11.6 Delete Hub Cluster

```bash id="c6"
eksctl delete cluster \
  --name hub-cluster \
  --region us-east-1
```

#### Important Note

Deleting the Hub Cluster will also remove:

* Argo CD control plane
* GitOps management layer
* Cluster registration data inside Kubernetes

### 11.7 Verify Cleanup

```bash id="c7"
eksctl get cluster
```

Expected Output:

```text id="c8"
No clusters found
```

### 11.8 Clean kubeconfig (Optional but Recommended)

Remove deleted cluster contexts:

```bash id="c9"
kubectl config get-contexts
kubectl config delete-context dev-cluster
kubectl config delete-context prod-cluster
kubectl config delete-context hub-cluster
```

### 11.9 AWS Console Verification

Manually verify in AWS Console:

* EKS → No clusters listed
* EC2 → No running worker nodes
* VPC → No leftover EKS VPCs (optional cleanup if needed)
* Load Balancers → Deleted automatically

### 11.10 Cost Safety Checklist

Ensure the following are removed:

| Resource          | Status     |
| ----------------- | ---------- |
| EKS Clusters      | Deleted    |
| EC2 Nodes         | Terminated |
| Load Balancers    | Removed    |
| VPC (EKS-created) | Cleaned    |
| Security Groups   | Removed    |

## Checkpoint

Before finishing this section, confirm:

* All 3 clusters are deleted
* `eksctl get cluster` shows no resources
* No EC2 instances are running
* No load balancers remain active
* kubeconfig is cleaned (optional but recommended)

### Expected Outcome

At the end of this section:

* Entire AWS infrastructure is successfully cleaned
* No active billing resources remain
* Kubernetes environment is fully torn down
* Lab lifecycle is completed end-to-end

### Key Learning (Important)

This step demonstrates a real-world DevOps principle:

> Infrastructure must be treated as **ephemeral and reproducible**

Using Infrastructure-as-Code + GitOps ensures:

* Easy rebuild of environments
* No dependency on manual setup
* Cost-controlled cloud usage
* Clean lifecycle management

### Lab Progress

| Component            | Status           |
| -------------------- | ---------------- |
| EKS Clusters         | ✅ Created        |
| GitOps Deployment    | ✅ Completed      |
| Drift Detection      | ✅ Completed      |
| Troubleshooting      | ✅ Completed      |
| Cleanup              | 🚀 Completed     |
| Infrastructure State | 🧹 Fully Deleted |

## 12. Real-World Use Cases & Industry Applications

### Objective

Understand how the **GitOps Multi-Cluster Hub–Spoke architecture using Argo CD on EKS** is used in real production environments.

This section connects your lab implementation to **enterprise-scale DevOps and Platform Engineering use cases**.

### 12.1 Why This Architecture Matters in Industry

Modern organizations do not run a single Kubernetes cluster. They operate:

* Multiple environments (Dev, QA, Stage, Prod)
* Multiple regions (US, EU, APAC)
* Multiple teams (microservices ownership)
* Strict compliance and isolation requirements

> The Hub–Spoke GitOps model solves this at scale.

### 12.2 Banking & Financial Systems

#### Use Case

Banks require:

* High availability
* Strong audit controls
* Environment isolation (Dev vs Prod)
* Zero manual deployment risk

#### How GitOps Helps

* Git acts as immutable audit log
* Argo CD ensures controlled deployments
* Multi-cluster separation improves compliance
* Drift detection prevents unauthorized changes

#### Example

* Dev cluster: feature testing
* Prod cluster: live transactions
* Hub cluster: centralized deployment governance

### 12.3 SaaS Platforms (Multi-Tenant Systems)

#### Use Case

SaaS companies manage:

* Thousands of deployments
* Multiple customers
* Frequent updates

#### Solution

* Each environment mapped to a cluster or namespace
* Git-based version control for releases
* Argo CD automates rollout across clusters

#### Outcome

* Faster releases
* Reduced human error
* Consistent deployments across tenants

### 12.4 Enterprise Platform Engineering

#### Use Case

Large enterprises build internal platforms for:

* Developer self-service
* Standardized infrastructure
* Centralized governance

#### GitOps Role

* Hub cluster acts as **platform control plane**
* Spoke clusters serve teams or departments
* Standardized deployment pipelines via Git

### 12.5 Dev/Test/Prod Environment Isolation

#### Use Case

Traditional CI/CD struggles with:

* Environment drift
* Manual deployment inconsistencies
* Configuration mismatch

#### GitOps Solution

| Environment | Cluster                  |
| ----------- | ------------------------ |
| Development | dev-cluster              |
| Testing     | stage-cluster (optional) |
| Production  | prod-cluster             |

Argo CD ensures:

* Same manifests across environments
* Controlled promotion via Git branches
* No manual deployment deviation

### 12.6 Multi-Region Kubernetes Architecture

#### Use Case

Global applications require:

* Low latency
* Disaster recovery
* Regional compliance

#### Architecture

* Hub cluster in primary region
* Spoke clusters in multiple regions
* Git as global control plane

#### Benefits

* Regional failover capability
* Independent scaling per region
* Centralized deployment governance

### 12.7 Microservices Platforms

#### Use Case

Microservices architectures involve:

* 10s to 100s of services
* Independent deployments
* Frequent updates

#### GitOps Advantage

* Each microservice = Git folder
* Argo CD manages deployment lifecycle
* Easy rollback using Git history

### 12.8 DevOps & SRE Teams

#### Use Case

SRE teams focus on:

* Reliability
* Automation
* Incident reduction

#### GitOps Benefits

* Automatic self-healing reduces incidents
* Drift detection improves system stability
* Declarative infrastructure reduces manual errors

### 12.9 CI/CD Modernization

#### Traditional CI/CD Problems

* Script-based deployments
* Manual rollback processes
* Environment inconsistencies

#### GitOps Improvement

* Git becomes deployment pipeline
* No direct cluster modifications
* Version-controlled infrastructure

### 12.10 Industry Tools Comparable to This Architecture

| Tool      | Role                        |
| --------- | --------------------------- |
| Argo CD   | GitOps controller           |
| FluxCD    | Alternative GitOps tool     |
| Terraform | Infrastructure provisioning |
| Helm      | Kubernetes packaging        |
| AWS EKS   | Managed Kubernetes platform |

### 12.11 Key Advantages of This Model

* Centralized control with distributed execution
* Strong security and access control
* Full auditability via Git
* Automated reconciliation
* Scalable multi-cluster management

### 12.12 Skills You Demonstrated

By completing this project, you have demonstrated:

* Kubernetes administration at scale
* AWS EKS cluster provisioning
* GitOps implementation using Argo CD
* Multi-cluster architecture design
* Infrastructure lifecycle management
* Continuous deployment automation
* Drift detection & self-healing systems
* Production-grade DevOps troubleshooting

### 12.13 Resume Impact (Important)

This project can be described as:

> Designed and implemented a GitOps-based multi-cluster Kubernetes deployment platform on AWS EKS using Argo CD, enabling centralized control, automated deployments, and self-healing infrastructure across Dev and Production environments.

## Final Summary

You have successfully built:

* A **Hub–Spoke Kubernetes architecture**
* A **production-style GitOps pipeline**
* A **multi-cluster Argo CD deployment system**
* A **self-healing infrastructure model**
* A **real-world DevOps platform simulation**

### Final Outcome

This project demonstrates **end-to-end DevOps engineering capability**, including:

* Cloud infrastructure provisioning (AWS EKS)
* Kubernetes orchestration
* GitOps automation
* Multi-cluster management
* Production-grade deployment strategies
