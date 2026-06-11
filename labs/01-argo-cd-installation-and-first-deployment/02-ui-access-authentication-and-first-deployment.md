
## 📄 `01-argo-cd-installation-and-first-deployment`

## Section 2: Accessing the UI, Authentication, First Login, and Deploying Your First Application

### Learning Objectives

After completing this section, you will be able to:

* Access the Argo CD UI
* Understand Argo CD networking requirements
* Expose Argo CD Server externally
* Retrieve the default admin password
* Login to Argo CD
* Understand authentication mechanisms
* Create your first Argo CD Application
* Deploy a Kubernetes application from Git
* Understand what an Argo CD Application actually is
* Observe GitOps synchronization in action

## 11. Accessing Argo CD After Installation

At the end of Section 1, Argo CD was installed successfully.

However, installation alone is not enough.

A common beginner question is:

> "I installed Argo CD. How do I access it?"

To answer this question, we first need to understand how Argo CD exposes its services.

### Understanding Argo CD Networking

When Argo CD is installed, Kubernetes creates several services.

Verify:

```bash
kubectl get svc -n argocd
```

Example output:

```text
NAME                     TYPE        CLUSTER-IP
argocd-server            ClusterIP
argocd-repo-server       ClusterIP
argocd-dex-server        ClusterIP
```

Notice:

```text
TYPE = ClusterIP
```

This is important.

### What is ClusterIP?

ClusterIP means:

```text
Accessible only inside Kubernetes
```

External users cannot access it directly.

#### Real-World Analogy

Imagine a corporate office.

Some rooms are internal only.

Visitors cannot enter them directly.

Instead, visitors must enter through reception.

ClusterIP services work similarly.

They are private services inside the cluster.

### Why Can't We Access the UI Yet?

Because:

```text
Browser
   │
   X
Cannot Reach
   │
ClusterIP Service
```

The service is internal.

Therefore, we need to expose it.

### Methods to Expose Argo CD

Several approaches exist.

| Method          | Suitable For |
| --------------- | ------------ |
| Port Forwarding | Learning     |
| NodePort        | Development  |
| LoadBalancer    | Cloud        |
| Ingress         | Production   |

For this lab we use:

```text
NodePort
```

because it is simple.

## 12. Converting Argo CD Server to NodePort

Locate the service:

```bash
kubectl get svc -n argocd
```

Find:

```text
argocd-server
```

Edit it:

```bash
kubectl edit svc argocd-server -n argocd
```

Locate:

```yaml
type: ClusterIP
```

Change to:

```yaml
type: NodePort
```

Save and exit.

### What Happens Internally?

Before:

```text
User
  │
  X
ClusterIP
```

After:

```text
User
  │
NodePort
  │
Argo CD Server
```

Now external traffic can reach Argo CD.

### Why Does Argo CD Server Need Exposure?

Remember from Section 1:

Argo CD Server acts as:

* UI endpoint
* CLI endpoint
* API endpoint

Without access to this service:

* UI won't work
* CLI won't work

## 13. Creating a Minikube Tunnel

For Minikube users, the easiest option is:

```bash
minikube service list -n argocd
```

Example:

```text
NAMESPACE   NAME             URL
argocd      argocd-server    https://127.0.0.1:56789
```

You can also directly access:

```bash
minikube service argocd-server -n argocd
```

Minikube automatically creates a tunnel.

### What Does Minikube Service Do?

Internally:

```text
Browser
   │
Local Tunnel
   │
NodePort
   │
Argo CD Server
```

> It creates a temporary route between your machine and the Kubernetes service.

### Accessing the UI

Open the URL returned by Minikube.

Example:

```text
https://127.0.0.1:56789
```

You may receive:

```text
Your connection is not private
```

This is expected.

Why?

> Because Argo CD generates self-signed certificates.

Choose:

```text
Advanced
Proceed Anyway
```

You should now see:

```text
Argo CD Login Screen
```

## 14. Understanding Authentication

Many beginners expect:

```text
Login with Google
Login with GitHub
Login with Azure
```

But they don't see those options.

Why?

> Because Dex has not yet been configured.

### Authentication Flow

Current setup:

```text
User
 │
 ▼
Local Admin Account
 │
 ▼
Argo CD
```

Enterprise setup:

```text
User
 │
 ▼
Google / Azure / Okta
 │
 ▼
Dex
 │
 ▼
Argo CD
```

### Understanding Dex Again

> Dex is the authentication broker.

Its responsibilities include:

* OAuth
* OIDC
* SSO

Without configuration, Dex remains unused.

### Real-World Analogy

Imagine airport security.

You can enter using:

* Passport
* Aadhaar
* Corporate ID

Dex acts as the officer validating those identities.

## 15. Retrieving the Initial Admin Password

Argo CD automatically creates an admin account.

Username:

```text
admin
```

Password:

Stored inside Kubernetes Secret.

View secrets:

```bash
kubectl get secrets -n argocd
```

You should find:

```text
argocd-initial-admin-secret
```

View secret:

```bash
kubectl edit secret argocd-initial-admin-secret -n argocd
```

Or:

```bash
kubectl get secret argocd-initial-admin-secret \
-n argocd -o yaml
```

Example:

```yaml
data:
  password: YWJjMTIzNDU=
```

The value is Base64 encoded.

### Decoding the Password

```bash
echo YWJjMTIzNDU= | base64 --decode
```

Output:

```text
abc12345
```

This becomes your login password.

### Why Are Secrets `Base64` Encoded?

Kubernetes stores secrets as:

```text
Base64 Encoded
```

> Important:

`Base64` is NOT encryption.

It is only encoding.

#### Interview Tip

Question:

"Is Kubernetes Secret secure because it is `Base64` encoded?"

Answer:

No.

> `Base64` provides no security.

Additional mechanisms like encryption-at-rest should be enabled.

### Logging In

Username:

```text
admin
```

Password:

```text
Decoded Password
```

Click:

```text
Login
```

> Congratulations.

You have successfully logged into Argo CD.

## 16. Understanding the Argo CD Dashboard

Initially, the dashboard appears empty.

Why?

Because Argo CD manages applications.

Currently:

```text
Applications = 0
```

No application has been registered yet.

### What is an Argo CD Application?

This is one of the most misunderstood concepts.

Many beginners assume:

```text
Application = Microservice
```

Not exactly.

### Argo CD Definition of Application

An Application is a Custom Resource.

It tells Argo CD:

```text
Which Git Repository?
Which Path?
Which Cluster?
Which Namespace?
```

Think of it as:

```text
Deployment Instructions
```

for Argo CD.

### Real-World Analogy

Suppose you tell a delivery company:

```text
Pick package from:
Warehouse A

Deliver to:
Building B
```

The instruction sheet is the Application.

Argo CD follows that instruction sheet.

### Application Architecture

```text
Application
      │
      ▼
Git Repository
      │
      ▼
Manifest Files
      │
      ▼
Kubernetes Cluster
```

## 17. Preparing Example Application

Instead of writing manifests ourselves, we use the official Argo CD examples repository.

Repository:

```text
https://github.com/argoproj/argocd-example-apps
```

This repository contains:

* Plain YAML examples
* Helm examples
* Kustomize examples

Perfect for learning.

### Guestbook Example

The simplest example:

```text
guestbook
```

Repository structure:

```text
guestbook
│
├── deployment.yaml
└── service.yaml
```

### Why Start With Guestbook?

Because it demonstrates:

* Deployment creation
* Service creation
* GitOps synchronization

without unnecessary complexity.

## 18. Creating the First Application

Click:

```text
New App
```

or

```text
Create Application
```

Fill:

#### Application Name

```text
my-first-app
```

#### Project

```text
default
```

#### Sync Policy

Select:

```text
Automatic
```

### What Does Automatic Mean?

Without automatic sync:

```text
Git Change
   │
   X
Manual Sync Required
```

With automatic sync:

```text
Git Change
   │
   ▼
Argo CD Detects
   │
   ▼
Deploys Automatically
```

This is GitOps.

### Source Configuration

Repository URL:

```text
https://github.com/argoproj/argocd-example-apps.git
```

Path:

```text
guestbook
```

Why Path?

Because one repository can contain:

```text
guestbook
helm-guestbook
kustomize-guestbook
multiple-apps
```

Argo CD needs to know which folder to deploy.

### Destination Configuration

Cluster:

```text
https://kubernetes.default.svc
```

This means:

```text
Current Kubernetes Cluster
```

Namespace:

```text
default
```

Click:

```text
Create
```

### What Happens Internally?

Immediately:

```text
Application Created
```

Then:

```text
Repo Server
     │
     ▼
Clone Repository
```

Then:

```text
Read guestbook Folder
```

Then:

```text
deployment.yaml
service.yaml
```

are parsed.

Then:

```text
Application Controller
```

creates resources inside Kubernetes.

### Internal Workflow

```text
GitHub
   │
   ▼
Repo Server
   │
   ▼
Application Controller
   │
   ▼
Kubernetes API
   │
   ▼
Deployment Created
Service Created
```

## 19. Verifying Deployment

Check:

```bash
kubectl get deploy
```

Expected:

```text
guestbook
```

Check Pods:

```bash
kubectl get pods
```

Expected:

```text
guestbook-xxxxx
```

Check Service:

```bash
kubectl get svc
```

Expected:

```text
guestbook
```

### Understanding What Just Happened

Notice something important.

You never executed:

```bash
kubectl apply
```

Yet:

```text
Deployment Created
Service Created
```

Who deployed them?

Answer:

```text
Argo CD
```

This is GitOps in action.

### Traditional CI/CD vs GitOps

Traditional:

```text
Git
 │
 ▼
Jenkins
 │
 ▼
Shell Script
 │
 ▼
kubectl apply
 │
 ▼
Cluster
```

GitOps:

```text
Git
 │
 ▼
Argo CD
 │
 ▼
Cluster
```

> Notice:

No deployment scripts.

No manual kubectl.

No SSH access.

### Benefits

* Simpler deployments
* Full audit trail
* Automatic rollback support
* Continuous reconciliation

## Section Summary

In this section we:

* ✓ Exposed Argo CD Server
* ✓ Accessed the UI
* ✓ Understood networking
* ✓ Created Minikube tunnel
* ✓ Retrieved admin credentials
* ✓ Learned authentication flow
* ✓ Understood Dex
* ✓ Logged into Argo CD
* ✓ Learned what an Application is
* ✓ Created first application
* ✓ Deployed Guestbook
* ✓ Observed GitOps deployment flow
* ✓ Verified deployment in Kubernetes

## Key Takeaways

1. Argo CD Server must be exposed before UI access.
2. Minikube service simplifies local access.
3. Default credentials are stored in Kubernetes Secrets.
4. Base64 is encoding, not encryption.
5. Applications are Argo CD Custom Resources.
6. Applications connect Git repositories to Kubernetes clusters.
7. Repo Server fetches manifests.
8. Application Controller deploys manifests.
9. Automatic Sync enables true GitOps.
10. Argo CD can deploy applications without manual kubectl commands.

## Interview Questions and Answers

#### 1. Why can't Argo CD UI be accessed immediately after installation?

**Answer:** Because the Argo CD Server service is created as a ClusterIP service and is only accessible within the cluster.

#### 2. What service provides the Argo CD UI?

**Answer:** `argocd-server`

#### 3. Where is the default admin password stored?

**Answer:** Inside the `argocd-initial-admin-secret` Kubernetes Secret.

#### 4. Is Base64 encryption?

**Answer:** No. Base64 is only an encoding mechanism.

#### 5. What is an Argo CD Application?

**Answer:** A Custom Resource that defines the source repository, deployment path, destination cluster, and target namespace.

#### 6. Which component clones Git repositories?

**Answer:** Repo Server.

#### 7. Which component deploys manifests into Kubernetes?

**Answer:** Application Controller.

#### 8. What happens when Automatic Sync is enabled?

**Answer:** Argo CD automatically deploys detected Git changes without manual synchronization.

#### 9. Why is Guestbook commonly used in demonstrations?

**Answer:** It is a simple application containing basic Kubernetes resources that make GitOps concepts easy to understand.

#### 10. What is the biggest difference between traditional CI/CD and GitOps?

**Answer:** GitOps continuously reconciles cluster state against Git, while traditional CI/CD usually performs one-time deployments through scripts or pipelines.

## End of Section 2

**Next Section:** *Understanding Continuous Reconciliation, Drift Detection, Sync Status, Application Health, Git Changes, Automatic Deployments, and Why Argo CD Is Not Opinionated (Plain YAML vs Helm vs Kustomize).*
