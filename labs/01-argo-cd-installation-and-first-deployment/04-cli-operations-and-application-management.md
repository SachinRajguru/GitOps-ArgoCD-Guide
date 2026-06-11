
## 📄 `01-argo-cd-installation-and-first-deployment`

## Section 4: Argo CD CLI Deep Dive — Installation, Authentication, Application Management, and UI vs CLI

### Learning Objectives

After completing this section, you will be able to:

* Install the Argo CD CLI
* Authenticate using the CLI
* Understand CLI architecture
* Create applications from the command line
* Synchronize applications
* Delete applications
* View application status
* Troubleshoot using CLI commands
* Compare UI and CLI workflows
* Understand when to use CLI instead of UI

## 30. Why Learn the Argo CD CLI?

Many engineers become comfortable with the UI.

However, in real production environments:

* UI access may be restricted
* VPN access may fail
* Load Balancer may be unavailable
* Bastion hosts may not support browsers

Yet deployments still need to happen.

This is where the CLI becomes extremely valuable.

### Real-World Scenario

Imagine:

Production Incident.

A deployment is:

```text
OutOfSync
```

You receive an alert at:

```text
2:00 AM
```

You connect through SSH to a jump server.

There is no browser.

There is no UI.

Only terminal access.

Would deployment management stop?

No.

The CLI allows full control of Argo CD.

### CLI vs UI Philosophy

Everything you perform in the UI:

```text
Create Application
Delete Application
Sync Application
View Status
```

can also be performed through the CLI.

### Architecture

UI Flow:

```text
User
 │
 ▼
Browser
 │
 ▼
Argo CD Server
```

CLI Flow:

```text
User
 │
 ▼
argocd CLI
 │
 ▼
Argo CD Server
```

Notice:

Both ultimately communicate with:

```text
Argo CD Server
```

## 31. Installing Argo CD CLI

The CLI is a separate binary.

> Installing Argo CD Server does NOT install the CLI.

### Installation on macOS

Using Homebrew:

```bash
brew install argocd
```

Verify:

```bash
argocd version
```

Example:

```text
argocd: v2.x.x
```

### Installation on Linux

Download binary:

```bash
curl -sSL -o argocd \
https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```

Make executable:

```bash
chmod +x argocd
```

Move:

```bash
sudo mv argocd /usr/local/bin/
```

Verify:

```bash
argocd version
```

### Installation on Windows

Using Chocolatey:

```powershell
choco install argocd-cli
```

Or download from GitHub releases.

### Why Separate CLI?

The CLI acts similarly to:

| Tool    | Purpose        |
| ------- | -------------- |
| kubectl | Kubernetes CLI |
| helm    | Helm CLI       |
| docker  | Docker CLI     |
| argocd  | Argo CD CLI    |

### Real-World Analogy

Think of Argo CD Server as a bank.

The CLI is your ATM card.

Different access method.

Same backend system.

## 32. Understanding CLI Authentication

Before creating applications, the CLI must authenticate.

Exactly like the UI.

### Common Beginner Mistake

Running:

```bash
argocd app create
```

without logging in.

Result:

```text
Permission Denied
Not Logged In
```

### Why Login Is Required

CLI must know:

```text
Which Argo CD Server?
Which Cluster?
Which User?
```

### Login Command

```bash
argocd login SERVER_URL
```

Example:

```bash
argocd login localhost:8080
```

### Authentication Flow

```text
CLI
 │
 ▼
Argo CD Server
 │
 ▼
Username Validation
 │
 ▼
Password Validation
 │
 ▼
Token Generation
```

### Example Login

```bash
argocd login localhost:8080
```

Prompt:

```text
Username:
```

Enter:

```text
admin
```

Prompt:

```text
Password:
```

Enter:

```text
Decoded Secret Password
```

Successful output:

```text
Login Succeeded
```

### What Happens Internally?

Argo CD generates:

```text
Authentication Token
```

and stores it locally.

Future commands use that token.

## 33. Verifying Login

Check account:

```bash
argocd account get-user-info
```

Output:

```text
Logged In User:
admin
```

Verify server:

```bash
argocd cluster list
```

Output:

```text
https://kubernetes.default.svc
```

This confirms:

```text
CLI ↔ Argo CD Server
```

communication works.

## 34. Exploring CLI Help

One of the most important skills:

Learning documentation.

Command:

```bash
argocd --help
```

Output:

```text
account
admin
app
cluster
context
login
logout
repo
version
```

Every Argo CD capability appears here.

### Example

Application-related commands:

```bash
argocd app --help
```

Output:

```text
create
delete
get
list
sync
wait
rollback
```

### Interview Tip

Strong DevOps engineers do not memorize every command.

They know how to discover commands quickly.

## 35. Creating Applications Using CLI

In Section 2 we created:

```text
Guestbook Application
```

through the UI.

Now we will create the same application using CLI.

### Equivalent UI Workflow

Previously:

```text
Create Application
Repository URL
Path
Namespace
Cluster
```

CLI simply converts those fields into parameters.

### Create Command

```bash
argocd app create guestbook \
--repo https://github.com/argoproj/argocd-example-apps.git \
--path guestbook \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
```

### Understanding Parameters

#### Application Name

```bash
guestbook
```

Creates:

```text
Application Object
```

inside Argo CD.

#### Repository

```bash
--repo
```

Specifies:

```text
Git Repository URL
```

#### Path

```bash
--path guestbook
```

Specifies:

```text
Which Folder?
```

inside the repository.

#### Namespace

```bash
--dest-namespace default
```

Specifies:

```text
Where To Deploy?
```

#### Destination Server

```bash
--dest-server
```

Specifies:

```text
Which Kubernetes Cluster?
```

### Internal Workflow

```text
CLI
 │
 ▼
Argo CD Server
 │
 ▼
Application Resource
 │
 ▼
Application Controller
 │
 ▼
Deployment
```

### Verify Application

List applications:

```bash
argocd app list
```

Example:

```text
NAME       STATUS
guestbook  Synced
```

### Detailed Information

```bash
argocd app get guestbook
```

Example:

```text
Name: guestbook

Sync Status:
Synced

Health:
Healthy
```

## 36. Synchronizing Applications

Sometimes Automatic Sync is disabled.

Scenario:

Developer updates Git.

Application shows:

```text
OutOfSync
```

Question:

How do we deploy manually?

### Sync Command

```bash
argocd app sync guestbook
```

Internal Workflow:

```text
Git Repository
      │
      ▼
Generate Manifest
      │
      ▼
Apply To Kubernetes
```

### Verify Sync

```bash
argocd app get guestbook
```

Result:

```text
Synced
Healthy
```

### Manual vs Automatic Sync

#### Manual

```text
Git Change
     │
     ▼
OutOfSync
     │
     ▼
Manual Sync
```

#### Automatic

```text
Git Change
     │
     ▼
Auto Detect
     │
     ▼
Auto Deploy
```

### Which Is Better?

Production:

Usually:

```text
Automatic Sync
```

Highly regulated environments:

Often:

```text
Manual Sync
```

to allow approvals.

## 37. Viewing Application Status

Quick overview:

```bash
argocd app list
```

Detailed view:

```bash
argocd app get guestbook
```

Resource-level view:

```bash
argocd app resources guestbook
```

Example:

```text
Deployment
Service
ReplicaSet
Pod
```

This helps troubleshoot.

## 38. Viewing Application History

Argo CD records deployment history.

Command:

```bash
argocd app history guestbook
```

Example:

```text
ID    DATE
1     Deployment
2     Deployment
3     Deployment
```

Useful for:

* Auditing
* Compliance
* Troubleshooting

## 39. Deleting Applications

Sometimes applications are no longer required.

Delete:

```bash
argocd app delete guestbook
```

Confirmation:

```text
Delete Application?
```

Answer:

```text
y
```

Result:

Application removed.

### What Gets Deleted?

Depends on configuration.

### Cascading Delete

Deletes:

```text
Application
Deployment
Service
Pods
```

### Non-Cascading Delete

Deletes:

```text
Application
```

Keeps:

```text
Deployment
Service
Pods
```

Advanced deletion policies are covered later.

## 40. Troubleshooting with CLI

One reason many engineers prefer CLI:

Fast troubleshooting.

### Check Application Status

```bash
argocd app get guestbook
```

### Check Synchronization

```bash
argocd app diff guestbook
```

Shows:

```text
Git
vs
Cluster
```

differences.

### Check Logs

Application controller:

```bash
kubectl logs deployment/argocd-application-controller -n argocd
```

Repo server:

```bash
kubectl logs deployment/argocd-repo-server -n argocd
```

Server:

```bash
kubectl logs deployment/argocd-server -n argocd
```

### Real Production Scenario

Application:

```text
OutOfSync
```

Reason:

```text
Deployment modified manually
```

Command:

```bash
argocd app diff guestbook
```

immediately reveals differences.

## 41. UI vs CLI Comparison

| Feature           | UI               | CLI          |
| ----------------- | ---------------- | ------------ |
| Beginner Friendly | Excellent        | Moderate     |
| Automation        | Limited          | Excellent    |
| Scripting         | No               | Yes          |
| CI/CD Integration | Limited          | Excellent    |
| Troubleshooting   | Good             | Excellent    |
| Bulk Operations   | Difficult        | Easy         |
| Remote Access     | Browser Required | SSH Friendly |
| Learning Curve    | Low              | Medium       |

### When Should You Use UI?

Use UI when:

* Learning Argo CD
* Visualizing resources
* Demonstrating deployments
* Reviewing health status

### When Should You Use CLI?

Use CLI when:

* Automating tasks
* Writing scripts
* Troubleshooting
* Managing production systems
* Working through SSH

### Real-World Best Practice

Most experienced DevOps engineers use:

```text
UI
for Visualization
```

and

```text
CLI
for Operations
```

### Example Enterprise Workflow

Deployment Monitoring:

```text
UI
```

Bulk Synchronization:

```text
CLI
```

Automated Deployment:

```text
CLI
```

Troubleshooting:

```text
CLI
```

## 42. Key CLI Commands Cheat Sheet

#### Login

```bash
argocd login SERVER
```

#### List Applications

```bash
argocd app list
```

#### Get Application

```bash
argocd app get APPNAME
```

#### Create Application

```bash
argocd app create
```

#### Sync Application

```bash
argocd app sync APPNAME
```

#### View Differences

```bash
argocd app diff APPNAME
```

#### View History

```bash
argocd app history APPNAME
```

#### Delete Application

```bash
argocd app delete APPNAME
```

#### Logout

```bash
argocd logout
```

## Chapter Summary

In this section we:

* ✓ Installed Argo CD CLI
* ✓ Learned CLI architecture
* ✓ Authenticated using CLI
* ✓ Created applications
* ✓ Listed applications
* ✓ Viewed application details
* ✓ Synchronized applications
* ✓ Viewed deployment history
* ✓ Deleted applications
* ✓ Learned troubleshooting commands
* ✓ Compared UI and CLI workflows
* ✓ Learned production best practices

## Key Takeaways

1. CLI communicates with Argo CD Server exactly like the UI.
2. Login is required before any CLI operation.
3. Applications can be fully managed through CLI.
4. CLI is essential for automation and production operations.
5. `argocd app create` is the CLI equivalent of Create Application in UI.
6. `argocd app sync` performs manual synchronization.
7. `argocd app diff` is extremely useful for troubleshooting.
8. UI is best for visualization.
9. CLI is best for operations and automation.
10. Strong DevOps engineers should be comfortable using both.


## Interview Questions

#### 1. Why would you use the Argo CD CLI instead of the UI?

**Answer:** For automation, scripting, troubleshooting, remote administration, and CI/CD integrations.

#### 2. Which component does the CLI communicate with?

**Answer:** Argo CD Server.

#### 3. How do you authenticate using the CLI?

```bash
argocd login SERVER_URL
```

#### 4. How can you view all applications?

```bash
argocd app list
```

#### 5. How do you manually synchronize an application?

```bash
argocd app sync APPNAME
```

#### 6. Which command shows Git vs Cluster differences?

```bash
argocd app diff APPNAME
```

#### 7. Can Argo CD be managed without the UI?

**Answer:** Yes. The CLI provides full functionality.

### 8. Which command shows application deployment history?

```bash
argocd app history APPNAME
```

#### 9. What is the CLI equivalent of creating an application through the UI?

**Answer:** `argocd app create`

#### 10. In enterprise environments, which is typically used more for automation: UI or CLI?

**Answer:** CLI.

## End of Section 4

**Next Section:** *Advanced Argo CD Configuration — Auto Sync, Self-Heal, Pruning, Ignore Differences, Application Projects, Multi-Cluster Management, and Enterprise GitOps Best Practices.*
