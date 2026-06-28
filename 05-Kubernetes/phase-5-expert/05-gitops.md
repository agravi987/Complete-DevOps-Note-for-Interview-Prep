# 🐙 GitOps (ArgoCD & Flux)

> **Hey Ravi! 👋** Think about how you've been deploying apps so far. You run `kubectl apply -f pod.yaml` from your laptop. What if your laptop dies? What if your coworker runs `kubectl edit` and changes a setting, but doesn't tell anyone? The cluster is now completely out of sync with your code repository! 😱 It's time to stop typing `kubectl apply` and let **GitOps** take over. 🤖

---

## 🧠 What is GitOps?

> **GitOps is an operational framework where a Git repository is the SINGLE source of truth for your cluster's desired state. An agent running in the cluster continuously pulls from Git and applies the YAML automatically.**

---

## 🏎️ Push vs Pull (The Paradigm Shift)

### The Old Way: Push (CI/CD Pipelines)
1. You push code to GitHub.
2. GitHub Actions / Jenkins (CI) builds the Docker image.
3. Jenkins runs `kubectl apply -f deploy.yaml` (CD).
*Problem:* You have to give Jenkins your highly secure cluster credentials! If Jenkins gets hacked, your cluster gets hacked. 🔓

### The GitOps Way: Pull (ArgoCD / Flux)
1. You push code to GitHub.
2. Jenkins builds the Docker image and updates the YAML file in Git with the new image tag.
3. **ArgoCD** (running INSIDE your secure cluster) is watching the Git repo.
4. ArgoCD sees the change, reaches out to Git, pulls the YAML, and applies it to the cluster itself.
*Benefit:* No external tool has access to your cluster! The cluster manages itself! 🔒

---

## 🐙 ArgoCD (The Industry Standard)

ArgoCD is the most popular GitOps tool because it comes with a beautiful UI that visually shows you the health of your apps! 🟢

**How it works:**
1. You create an `Application` CRD in Kubernetes.
2. You tell it: "Watch this GitHub URL, branch `main`, folder `production/`".
3. ArgoCD continuously runs a reconciliation loop.
4. If a developer manually runs `kubectl scale deploy my-app --replicas=10`, ArgoCD sees that Git still says `replicas: 3`. ArgoCD will immediately overwrite the developer's change to force reality back to Git's desired state! 👮‍♂️

### 🧩 The App of Apps Pattern
How do you deploy ArgoCD itself? How do you deploy 50 apps?
You create ONE "Root Application" that points to a Git folder containing 50 other `Application` YAMLs. ArgoCD reads the root app, spawns 50 child apps, and those apps spawn your deployments! It's Argoception! 🤯

---

## ⏪ Rollbacks in GitOps

In the old days, you ran `helm rollback` or `kubectl rollout undo`. 
In GitOps, you NEVER touch the cluster to roll back.

**How to rollback in GitOps:**
You simply run `git revert` on the bad commit and push to `main`! ArgoCD sees the old state in Git, and synchronizes the cluster back to safety. Git is your absolute audit log.

---

## 🔧 Image Updater (The Missing Link)

If developers push new code, who updates the Git YAML repository with the new Docker image tag (`v1.0.1` -> `v1.0.2`)?
1. Your CI pipeline can run a `git commit` to do it.
2. Or, you can use **ArgoCD Image Updater**, which watches your Docker Registry directly! When a new image is pushed, the Updater automatically commits the new tag to your Git repo for you! 🤖✍️

---

## 🎤 Interview Knockout Answers

**Q: Explain the concept of GitOps and its main benefits.**
> "GitOps is an operating model where a Git repository acts as the single source of truth for declarative infrastructure and applications. Instead of pushing changes via external CI/CD pipelines, a software agent running inside the cluster (like ArgoCD or Flux) continuously pulls the desired state from Git and reconciles the cluster to match it. Benefits include enhanced security (no external access required), clear audit trails via Git commit history, and trivial rollbacks via `git revert`."

**Q: In a GitOps workflow, how do you prevent configuration drift?**
> "Configuration drift happens when someone manually alters the cluster state (e.g., using `kubectl edit`) so it no longer matches the Git repository. GitOps tools like ArgoCD detect this drift instantly during their reconciliation loop. If 'Auto-Sync' is enabled, ArgoCD will immediately overwrite the manual changes to enforce the state declared in Git, ensuring drift is automatically corrected."

**Q: What is the 'App of Apps' pattern in ArgoCD?**
> "The App of Apps pattern is a declarative way to manage a whole fleet of applications. Instead of creating ArgoCD Application objects manually, you create a single 'Root' Application that points to a Git directory containing the YAML definitions for all your other Applications. ArgoCD syncs the Root, which in turn automatically syncs all the child apps. It's highly scalable for bootstrapping entire clusters."

---

## ⚡ 30-Second Revision

- 🐙 **GitOps** = Git is the only source of truth. No manual `kubectl apply`.
- 🏎️ **Pull Model** = Agent inside cluster pulls from Git (Highly secure!).
- 👮‍♂️ **Drift Protection** = Overwrites any manual changes made to the cluster.
- ⏪ **Rollbacks** = Just use `git revert`.
- 📊 **ArgoCD** = Has a great UI. **Flux** = CLI focused, fully automated.
- 🧩 **App of Apps** = One Argo app that deploys all your other Argo apps.

> Next → [HA & Disaster Recovery](06-ha-and-disaster-recovery.md) 🚀
