# 🤖 Service Accounts — Identity for Your Apps

> **Hey Ravi! 👋** We just talked about RBAC for humans (like you and me). But what if your Python script inside a Pod needs to call the Kubernetes API to check how many nodes are running? Or what if your Pod needs to talk to AWS to upload a file to S3? Your Pod needs an ID card! 🪪 That's what **Service Accounts** are for.

---

## 🧠 What is a Service Account?

> **A ServiceAccount (SA) is an identity given to a Pod, allowing the application running inside the Pod to authenticate with the Kubernetes API (or external cloud providers).**

- **Users** = Humans (Ravi, Admin).
- **ServiceAccounts** = Robots (Pods, CI/CD pipelines, Prometheus).

---

## 💳 The Corporate ID Card Analogy

When a human employee enters the building, they swipe their personal ID badge.
When a security camera (a robot) connects to the network, it doesn't use a human's name; it has a "Service Account" configured in its software with specific, limited permissions (like "upload video only").

In K8s, the Pod is the robot, and the ServiceAccount is its ID card! 🪪

---

## 🏗️ The `default` Service Account

If you don't specify a ServiceAccount in your Pod YAML, Kubernetes injects one automatically. 
Every namespace automatically has a ServiceAccount literally named `default`. 

**The Danger:** 🚨 By default, Kubernetes mounts a secret token for this `default` account into EVERY pod at `/var/run/secrets/kubernetes.io/serviceaccount`. If a hacker compromises your pod, they get that token!

> **Best Practice:** ALWAYS set `automountServiceAccountToken: false` on pods that don't need to talk to the K8s API!

---

## 📄 Real YAML (Assigning an SA to a Pod)

### 1. Create the ServiceAccount
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-reader-sa
  namespace: prod
```
*(Note: You would then create a Role and RoleBinding to give `api-reader-sa` permissions!)*

### 2. Attach it to the Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  serviceAccountName: api-reader-sa    # Uses our custom SA!
  # automountServiceAccountToken: false  # (Use this if you DON'T want the token mounted!)
  containers:
  - name: app
    image: my-app:v1
```

---

## ☁️ IRSA: IAM Roles for Service Accounts (AWS EKS)

This is a MASSIVE interview topic for Cloud/DevOps engineers! ☁️

Historically, if a Pod needed to talk to AWS S3, you had to give the underlying Worker Node permission to talk to S3. But then ALL pods on that node could access S3! Huge security risk. 😱

**IRSA solves this:**
You map a Kubernetes `ServiceAccount` directly to an AWS `IAM Role`. 
Only the Pod using that specific ServiceAccount gets the AWS permissions!

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-uploader-sa
  annotations:
    # MAGIC! This links the K8s SA to an AWS IAM Role! 🪄
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/my-s3-role
```
*(GCP has the exact same concept called "Workload Identity").*

---

## 🎮 Commands

```bash
# List ServiceAccounts in namespace
kubectl get sa

# Look at the details of an SA
kubectl describe sa api-reader-sa

# Manually create a token for a ServiceAccount (used for CI/CD pipelines)
kubectl create token api-reader-sa --duration=1h
```

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between a User and a ServiceAccount?**
> "Users are meant for humans. Kubernetes doesn't actually manage Users internally (it relies on external systems like OIDC, LDAP, or AWS IAM). ServiceAccounts are meant for processes running inside Pods. Kubernetes manages ServiceAccounts internally, creating them as API objects and automatically mounting their authentication tokens into Pods."

**Q: Explain IRSA (IAM Roles for Service Accounts) in EKS.**
> "IRSA allows you to assign AWS IAM permissions at the Pod level rather than the Node level. You annotate a Kubernetes ServiceAccount with an AWS IAM Role ARN. When a Pod uses that ServiceAccount, the AWS SDK inside the container automatically retrieves temporary STS credentials for that IAM role, ensuring the principle of least privilege."

**Q: Why is the `default` ServiceAccount considered a security risk?**
> "Every namespace has a `default` ServiceAccount, and K8s automatically mounts its token into every Pod by default. If a Pod is compromised (e.g., via Remote Code Execution), an attacker can use that token to interact with the Kubernetes API. You should always set `automountServiceAccountToken: false` on pods that don't need API access."

---

## ⚡ 30-Second Revision

- 🤖 **ServiceAccount (SA)** = Identity for your Pods.
- 🪪 Automatically mounted as a file token inside the container.
- 🚫 Set `automountServiceAccountToken: false` for security if API access isn't needed.
- 🖇️ You use RBAC (RoleBindings) to give an SA permissions.
- ☁️ **IRSA / Workload Identity** = Linking a K8s SA to a Cloud IAM Role (AWS/GCP). Highly secure!

> Next → [Network Policies](04-network-policies.md) 🚀
