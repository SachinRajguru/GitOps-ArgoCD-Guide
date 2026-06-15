
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

Before accessing the UI, we must first understand how Argo CD exposes its services inside Kubernetes.

### Verifying Installed Services

When Argo CD is installed, Kubernetes creates several services.

Run:

```bash
kubectl get svc -n argocd
```

Example output:

```text
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)
argocd-server        ClusterIP   10.96.112.31   <none>        80/TCP,443/TCP
argocd-repo-server   ClusterIP   10.96.75.189   <none>        8081/TCP
argocd-dex-server    ClusterIP   10.96.92.87    <none>        5556/TCP,5557/TCP
argocd-redis         ClusterIP   10.96.118.43   <none>        6379/TCP
```

Notice:

```text
TYPE = ClusterIP
```

This is important because it determines who can access the service.

### What is ClusterIP?

ClusterIP means:

```text
Accessible only from inside the Kubernetes cluster network
```

External users cannot access it directly.

#### Real-World Analogy

Think of a corporate office:
   * Internal rooms are not accessible to visitors
   * Only employees inside the office can enter

ClusterIP works the same way:
   * It keeps services private inside the cluster.

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

The service is internal to Kubernetes.

Therefore, we need to expose it.

### Methods to Expose Argo CD

Several approaches exist:

| Method          | Suitable For       | Pros                 | Cons                        |
|-----------------|--------------------|----------------------|-----------------------------|
| Port Forwarding | Learning / Testing | Simple, no config    | Temporary, manual           |
| NodePort        | Development        | Easy setup           | Limited port range          |
| LoadBalancer    | Cloud Production   | Full external access | Requires cloud LB           |
| Ingress         | Production         | Flexible routing     | Requires ingress controller |

For this lab we use:

* NodePort (general Kubernetes approach)
* Minikube Service (Minikube-specific shortcut)

## 12. Exposing Argo CD Using NodePort

One common way to expose Argo CD is to convert the `argocd-server` service from a **ClusterIP** service into a **NodePort** service.

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

## 13. Accessing Argo CD Using Minikube Service

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
It creates a temporary route between your machine and the Kubernetes service.

> It creates a temporary tunnel between your machine and the Kubernetes service.

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

* Login with Google
* Login with GitHub
* Login with Azure

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
* SAML
* SSO integrations

> Without configuration, Dex remains unused.

### Real-World Analogy

Imagine airport security.

You can enter using:

* Passport
* Government ID   
* Corporate Badge

Dex acts as the officer validating those identities.

## 15. Retrieving the Initial Admin Password

Argo CD automatically creates an admin account.

**Username:**

```text
admin
```

**Password:**

Stored inside Kubernetes Secret.

**View secrets:**

```bash
kubectl get secrets -n argocd
```

output:

```text
NAME                          TYPE     DATA   AGE
argocd-initial-admin-secret   Opaque   1      5m
argocd-secret                 Opaque   1      5m
argocd-repo-server            Opaque   1      5m
```

You should find:

```text
argocd-initial-admin-secret
```

**View secret:**

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

> **Important:** The password shown above is an example. Use the actual value from your cluster.

### Why Are Secrets Base64 Encoded?

Kubernetes stores secrets as:

```text
Base64 Encoded
```

Important:

Base64 is NOT encryption.

It is only encoding.

> #### Interview Tip

Question:

"Is Kubernetes Secret secure because it is Base64 encoded?"

Answer:

No.

Base64 provides no security.

Additional mechanisms like encryption-at-rest should be enabled.

> Base64 is easily decoded. For production, enable encryption-at-rest for `etcd` and use external secrets management solutions like `Vault` or `AWS Secrets Manager`.

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

You have successfully logged into [Argo CD](https://argoproj.github.io/cd/).

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

An Application is a **Custom Resource Definition (CRD)**.

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
Pick package from: Warehouse A

Deliver to: Building B
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

> Instead of writing manifests ourselves, we use the official [Argo CD examples](https://github.com/argoproj/argocd-example-apps) repository.

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

In the Argo CD UI:

Click:

```text
New App
```

or

```text
Create Application
```

Fill in the following details::

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
NAME        READY   UP-TO-DATE   AVAILABLE
guestbook   1/1     1            1
```

Check Pods:

```bash
kubectl get pods
```

Expected:

```text
NAME                        READY   STATUS
guestbook-7d9c9b5c8-abc12   1/1     Running
```

Check Service:

```bash
kubectl get svc guestbook
```

Expected:

```text
NAME        TYPE       CLUSTER-IP    PORT(S)
guestbook   ClusterIP  10.108.x.x    80:xxxxx/TCP
```

### Understanding What Just Happened

Notice something important.

You never executed:

```bash
kubectl apply
```

Yet:

```text
Deployment Created ✓
Service Created ✓
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

Notice:

No deployment scripts.

No manual `kubectl`.

No SSH access.

### Benefits

* Simpler deployments
* Full audit trail
* Automatic rollback support
* Continuous reconciliation

## Section Summary

In this section we:

* ✓ Exposed Argo CD Server using NodePort
* ✓ Accessed the UI via Minikube service
* ✓ Understood Kubernetes networking (ClusterIP vs NodePort)
* ✓ Created Minikube tunnel
* ✓ Retrieved admin credentials from Kubernetes Secrets
* ✓ Learned authentication flow (local vs enterprise)
* ✓ Understood Dex role in authentication
* ✓ Logged into Argo CD
* ✓ Learned what an Argo CD Application is
* ✓ Created first Application via UI
* ✓ Deployed Guestbook application
* ✓ Observed GitOps deployment flow
* ✓ Verified deployment in Kubernetes
* ✓ Compared Traditional CI/CD vs GitOps

## Key Takeaways

1. **ClusterIP** services are internal only; **NodePort** exposes them externally
2. Argo CD Server must be exposed before UI access
3. Minikube service simplifies local access automatically
4. Default credentials are stored in Kubernetes Secrets as Base64-encoded values
5. Base64 is encoding, NOT encryption
6. **Argo CD Application** is a Custom Resource that defines deployment instructions
7. Applications connect Git repositories to Kubernetes clusters
8. **Repo Server** fetches manifests from Git
9. **Application Controller** deploys manifests to Kubernetes
10. **Automatic Sync** enables true GitOps — Git changes trigger automatic deployments

## Interview Questions and Answers

### 1. Why can't Argo CD UI be accessed immediately after installation?

**Answer:** Because the Argo CD Server service is created as a ClusterIP service and is only accessible within the cluster. External access requires exposing the service via NodePort, LoadBalancer, or Ingress.

### 2. What service provides the Argo CD UI?

**Answer:** `argocd-server`

### 3. Where is the default admin password stored?

**Answer:** Inside the `argocd-initial-admin-secret` Kubernetes Secret in the `argocd` namespace.

### 4. Is Base64 encryption?

**Answer:** No. Base64 is only an encoding mechanism used for data transportation. It provides no security. Kubernetes Secrets should be protected with encryption-at-rest in production.

### 5. What is an Argo CD Application?

**Answer:** A Custom Resource Definition (CRD) that defines where to find manifests (Git repository), which path to use, which cluster to deploy to, and which namespace to target.

### 6. Which component clones Git repositories?

**Answer:** Repo Server (`argocd-repo-server`)

### 7. Which component deploys manifests into Kubernetes?

**Answer:** Application Controller (`argocd-application-controller`)

### 8. What happens when Automatic Sync is enabled?

**Answer:** Argo CD automatically detects changes in Git and deploys them to the cluster without manual intervention (synchronization), enabling true GitOps. 

### 9. Why is Guestbook commonly used in demonstrations?

**Answer:** It is a simple application containing basic Kubernetes resources (Deployment and Service) that make GitOps concepts easy to understand without unnecessary complexity.

### 10. What is the biggest difference between traditional CI/CD and GitOps?

**Answer:** Traditional CI/CD uses external pipelines (Jenkins, GitLab CI) to run kubectl apply, while GitOps uses a controller inside the cluster (Argo CD) that continuously reconciles cluster state against Git. GitOps shifts the source of truth to Git and provides continuous reconciliation.

### Question 11: How do you expose Argo CD Server externally?

**Answer:**

```bash
# Method 1: Change service type to NodePort
kubectl patch svc argocd-server -n argocd \
-p '{"spec":{"type":"NodePort"}}'
```
```bash
# Method 2: Use Minikube service
minikube service argocd-server -n argocd
```

###  Question 12: What is the difference between ClusterIP, NodePort, and LoadBalancer?

**Answer:**

| Type         |  Accessibility            |  Use Case                  | 
|--------------|---------------------------|----------------------------|
| ClusterIP    | Internal only             | Microservice communication |
| NodePort     | External via node IP:port | Development/testing        |
| LoadBalancer | External via cloud LB     | Production (cloud)         |

### Question 13: How do you retrieve the initial admin password?

**Answer:**

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

### Question 14: What happens if Git and Kubernetes states differ?

**Answer:** Argo CD marks the Application as **OutOfSync** and can either alert or automatically reconcile depending on sync policy.

### Question 15: Can Argo CD deploy applications without manual kubectl commands?

**Answer:** Yes. This is the core benefit of GitOps. Argo CD continuously watches Git and applies changes automatically when Automatic Sync is enabled.

## End of Section 2

**Next Section:** *Understanding Continuous Reconciliation, Drift Detection, Sync Status, Application Health, Git Changes, Automatic Deployments, and Why Argo CD Is Not Opinionated (Plain YAML vs Helm vs Kustomize).*
