
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

* Create Application
* Delete Application
* Sync Application
* View Status

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

Notice: Both ultimately communicate with **Argo CD Server**

## 31. Installing Argo CD CLI

The CLI is a separate binary.

> Installing Argo CD Server does NOT install the CLI.

### 31.1 Installation on macOS

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

### 31.2 Installation on Linux

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

### 31.3 Installation on Windows

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

* Which Argo CD Server?
* Which Cluster?
* Which User?

### Login Command

```bash
argocd login <server>
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
Logged In User: admin
Email: admin@example.com
```

Verify server connection:

```bash
argocd cluster list
```

Output:

```text
SERVER                          NAME        VERSION
https://kubernetes.default.svc  in-cluster  1.28+
```

This confirms CLI ↔ Argo CD Server communication works.

## 34. Exploring CLI Help

One of the most important DevOps skills is knowing how to navigate documentation efficiently.

Argo CD provides built-in help for every command.

Run:

```bash
argocd --help
```

This displays the available command groups and global options.

### Exploring Subcommands

To view commands related to application management:

```bash
argocd app --help
```

Similarly, you can explore other command groups:

```bash
argocd cluster --help
argocd repo --help
argocd account --help
```

### Official Command Reference

For the complete and up-to-date command reference, visit:

https://argo-cd.readthedocs.io/en/latest/user-guide/commands/argocd/

The documentation includes:

* Command syntax
* Available options
* Examples
* Subcommands
* Version-specific updates

### Interview Tip

Strong DevOps engineers do not memorize every command, flag, or option.

Instead, they know how to quickly discover commands using:

```bash
argocd --help
```

and consult the official documentation when needed.

## 35. Creating Applications Using CLI

In Section 2, we created the **Guestbook Application** through the UI.

Now we will create the same application using CLI.

### UI vs CLI Equivalence

| UI Field         | CLI Parameter      |
|------------------|--------------------|
| Application Name | `<app-name>`       |
| Repository URL   | `--repo`           |
| Path             | `--path`           |
| Namespace        | `--dest-namespace` |
| Cluster          | `--dest-server`    |

### Create Command

```bash
argocd app create guestbook \
--repo https://github.com/argoproj/argocd-example-apps.git \
--path guestbook \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
```

### Understanding Parameters

| Parameter          | Value            | Description                                    |
|--------------------|------------------|------------------------------------------------|
| `guestbook`        | Application name | Creates an Argo CD Application resource (CR)   |
| `--repo`           | Git URL          | Git repository location                        |
| `--path`           | `guestbook`      | Folder path inside the repository              |
| `--dest-namespace` | `default`        | Target namespace                               |
| `--dest-server`    | Cluster URL      | Target Kubernetes cluster                      |

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
The CLI sends the request to the Argo CD API server, which creates an Application custom resource. The Application Controller then monitors the resource and performs the deployment.

### Verify Application

List applications:

```bash
argocd app list
```

Example:

```text
NAME       STATUS   HEALTH
guestbook  Synced   Healthy
```

> Actual output may vary slightly depending on the Argo CD version.

### Get Detailed Information

```bash
argocd app get guestbook
```

Example Output:

```text
Name:               guestbook
Server:             https://kubernetes.default.svc
Namespace:          default
Repo:               https://github.com/argoproj/argocd-example-apps.git
Path:               guestbook
Sync Policy:        Manual
Sync Status:        Synced
Health Status:      Healthy
```

## 36. Synchronizing Applications

Sometimes Automatic Sync is disabled.

**Scenario:**

Developer updates Git.

Application shows:

```text
OutOfSync
```

**Question:**

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
Status:  Synced
Health:  Healthy
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

| Environment      | Recommended                  |
|------------------|------------------------------|
| Production       | Usually Automatic Sync       |
| Highly Regulated | Manual Sync (approval-based) |

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
GROUP   KIND        NAMESPACE  NAME       STATUS
apps    Deployment  default    guestbook  Synced
        Service     default    guestbook  Synced
```

This helps with troubleshooting.

## 38. Viewing Application History

Argo CD records deployment history.

Command:

```bash
argocd app history guestbook
```

Example:

```text
ID  DATE               COMMIT               SYNC
1   2026-06-01 10:00   First deployment     Synced
2   2026-06-02 14:30   Update replicas      Synced
3   2026-06-03 09:15   Fix image tag        Synced
```

Useful for:

* Auditing
* Compliance
* Troubleshooting
* Rollback identification

## 39. Deleting Applications

Sometimes applications are no longer required.

Delete:

```bash
argocd app delete guestbook
```

Confirmation:

```text
Delete application 'guestbook'? [y/N]
```

Answer:

```text
y
```

Result:

Application removed.

### What Gets Deleted?

The behavior depends on the deletion policy.

### Cascading Delete

Default behavior: Deletes both Application AND deployed resources.

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

To keep deployed resources:

```bash
argocd app delete guestbook \
--cascade=false
```

Result:

```text
Application CR deleted
Deployment retained
```

> Advanced deletion policies and resource finalizers are covered later.

## 40. Troubleshooting with CLI

One reason many engineers prefer CLI: Fast troubleshooting.

### Check Application Status

```bash
argocd app get guestbook
```

### Check Synchronization

```bash
argocd app diff guestbook
```

Shows differences between:

```text
Git (desired state)
vs
Cluster (live state)
```

Example output:

```text
--- deployment.yaml
+++ cluster state
@@ -3,7 +3,7 @@
 spec:
-  replicas: 3
+  replicas: 10
```

### Check Logs

Application controller:

```bash
kubectl logs deployment/argocd-application-controller -n argocd
```

Repo server:

```bash
kubectl logs deployment/argocd-repo-server -n argocd
```

API Server:

```bash
kubectl logs deployment/argocd-server -n argocd
```

### Real Production Scenario

**Problem:** Application shows OutOfSync

Application:

```text
OutOfSync
```

Possible reason:

```text
Deployment modified manually
```

Command:

```bash
argocd app diff guestbook
```

Result: 

The command immediately highlights differences between Git and the live cluster state, helping identify manual drift.

## 41. UI vs CLI Comparison

| Feature           | UI               | CLI                    |
| ----------------- | ---------------- | ---------------------- |
| Beginner Friendly | Excellent        | Moderate               |
| Automation        | Basic            | Excellent              |
| Scripting         | No               | Yes                    |
| CI/CD Integration | Basic            | Excellent              |
| Troubleshooting   | Good             | Excellent              |
| Bulk Operations   | Difficult        | Easy                   |
| Remote Access     | Browser Required | Terminal/SSH Friendly  |
| Learning Curve    | Low              | Medium                 |
| Visual Feedback   | Rich             | Text-based             |

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

Most DevOps teams use both interfaces together:

| Tool | Purpose                          |
|------|----------------------------------|
| UI   | For visualization and monitoring |
| CLI  | For operations and automation    |

### Example Enterprise Workflow

| Task                  | Tool |
|-----------------------|------|
| Deployment Monitoring | UI   |
| Bulk Synchronization  | CLI  |
| Automated Deployment  | CLI  |
| Troubleshooting       | CLI  |

## 42. Key CLI Commands Cheat Sheet

#### Login & Logout

```bash
# Login to Argo CD server
argocd login <server>      # e.g.: argocd login localhost:8080
```
```bash
# Logout
argocd logout
```

#### Application Management

```bash
# List all applications
argocd app list
```
```bash
# Get application details
argocd app get guestbook
```
```bash
# Create application
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-namespace default \
  --dest-server https://kubernetes.default.svc
```
```bash
# Sync application
argocd app sync guestbook
```
```bash
# View differences
argocd app diff guestbook
```
```bash
# View history
argocd app history guestbook
```
```bash
# Delete application
argocd app delete guestbook
```

#### Utility Commands

```bash
# Check version
argocd version
```
```bash
# View cluster list
argocd cluster list
```
```bash
# View help
argocd --help
argocd app --help
```

## Section Summary

In this section we:

* ✓ Installed Argo CD CLI on macOS, Linux, and Windows
* ✓ Understood CLI architecture
* ✓ Learned CLI authentication flow
* ✓ Verified login and server connection
* ✓ Explored CLI help commands
* ✓ Created applications using CLI
* ✓ Listed and viewed application details
* ✓ Synchronized applications manually
* ✓ Viewed deployment history
* ✓ Deleted applications
* ✓ Learned troubleshooting commands
* ✓ Compared UI and CLI workflows
* ✓ Understood production best practices

## Key Takeaways

1. CLI communicates with **Argo CD Server** exactly like the UI.
2. **Login is required** before any CLI operation.
3. Applications can be fully managed through CLI — there is nothing the UI can do that CLI cannot.
4. CLI is essential for **automation** and **production** operations.
5. `argocd app create` is the CLI equivalent of "Create Application" in UI.
6. `argocd app sync` performs manual synchronization.
7. `argocd app diff` is extremely useful for **troubleshooting**.
8. **UI** is best for visualization.
9. **CLI** is best for operations and automation.
10. Strong DevOps engineers should be comfortable using **both**.

## Interview Questions

#### 1. Why would you use the Argo CD CLI instead of the UI?

**Answer:** For automation, scripting, troubleshooting, remote administration via SSH, and CI/CD integrations. CLI is essential when browser access is not available.

#### 2. Which component does the CLI communicate with?

**Answer:** Argo CD Server (`argocd-server`).

#### 3. How do you authenticate using the CLI?

```bash
argocd login <server-url>
```

Example:

```bash
argocd login localhost:8080
```

#### 4. How can you view all applications?

```bash
argocd app list
```

#### 5. How do you manually synchronize an application?

```bash
argocd app sync <app-name>
```

Example:

```bash
argocd app sync guestbook
```

#### 6. Which command shows Git vs Cluster differences?

```bash
argocd app diff <app-name>
```

This command is extremely useful for troubleshooting configuration drift.

#### 7. Can Argo CD be managed without the UI?

**Answer:** Yes. The CLI provides full functionality. Every operation available in the UI can be performed via CLI.

### 8. Which command shows application deployment history?

**Answer:**

```bash
argocd app history <app-name>
```

Example:

```bash
argocd app history guestbook
```

#### 9. What is the CLI equivalent of creating an application through the UI?

**Answer:** 

```bash
argocd app create <app-name> \
  --repo <repository-url> \
  --path <path-in-repository> \
  --dest-namespace <namespace> \
  --dest-server <cluster-url>
```

#### 10. In enterprise environments, which is typically used more for automation: UI or CLI?

**Answer:** CLI. 

The CLI is preferred for:

* CI/CD pipelines
* Scripted operations
* Bulk management
* Automation

## End of Section 4

**Next Section:** *Advanced Argo CD Configuration — Auto Sync, Self-Heal, Pruning, Ignore Differences, Application Projects, Multi-Cluster Management, and Enterprise GitOps Best Practices.*
