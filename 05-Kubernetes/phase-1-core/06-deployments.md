# 🚀 Deployments — The Production King

> **Hey Ravi! 👋** If you only learn ONE Kubernetes object deeply — make it **Deployments.** They are what you use 90% of the time in real production. Everything runs through Deployments. Let's master this! 👑

---

## 🧠 What is a Deployment?

> **A Deployment manages your app's desired state AND handles updates smoothly, with full rollback capability.**

It does 3 things that make it king:

| What it does | Why you care |
|---|---|
| 🔁 Keeps pods alive (like ReplicaSet) | High availability |
| 🚀 Rolls out updates safely | Zero downtime deploys |
| ⏪ Rolls back if something breaks | Sleep at night peacefully |

---

## 🍽️ The Restaurant Menu Analogy

> You run a restaurant and want to update the pasta recipe WITHOUT shutting down the restaurant.

```
Old recipe (v1) → 5 chefs cooking it
New recipe (v2) → introduce slowly

Step 1: Hire 1 chef with new recipe (1 new pod up)
Step 2: Fire 1 chef with old recipe (1 old pod down)
Step 3: Repeat until all chefs use new recipe
Step 4: If new recipe tastes bad → rollback! Everyone goes back to v1

→ Customers never stopped eating. Zero downtime! 🎉
```

That's a **rolling update.** That's what Deployments do.

---

## 🏗️ How Deployments Really Work

```
┌─────────────────────────────────────────┐
│             Deployment                  │
│                                         │
│   ┌──────────────────────────────┐      │
│   │  ReplicaSet v1 (old)         │      │
│   │  ███░░░  (3 pods)            │      │
│   └──────────────────────────────┘      │
│                                         │
│   ┌──────────────────────────────┐      │
│   │  ReplicaSet v2 (new) ←──────┼──────┤← update!
│   │  ██████  (3 pods)            │      │
│   └──────────────────────────────┘      │
└─────────────────────────────────────────┘
```

**During a rolling update:** Deployment spins up new ReplicaSet (v2) and scales down old ReplicaSet (v1) gradually. Old pods → new pods, one at a time!

**After rollback:** Deployment scales v2 back to 0, scales v1 back up. Boom. Like it never happened.

---

## 📄 Deployment YAML — Full Explained

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app           # Deployment name
  labels:
    app: web-app

spec:
  replicas: 3             # I want 3 pods ALWAYS

  selector:
    matchLabels:
      app: web-app        # "Own pods with this label"

  strategy:
    type: RollingUpdate   # How to do updates
    rollingUpdate:
      maxSurge: 1         # Create 1 extra pod during update
      maxUnavailable: 1   # Allow 1 pod to be unavailable during update

  template:               # Pod blueprint
    metadata:
      labels:
        app: web-app      # MUST match selector!
    spec:
      containers:
      - name: web
        image: nginx:1.27  # ← Change this to trigger a rollout!
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

---

## 🎮 Deployment Commands — The Ones You'll Use Daily

```bash
# Create/update deployment
kubectl apply -f deployment.yaml

# See deployment status
kubectl get deployment web-app

# Watch rollout in real-time
kubectl rollout status deployment/web-app

# TRIGGER A NEW ROLLOUT (update the image)
kubectl set image deployment/web-app web=nginx:1.28

# See rollout history
kubectl rollout history deployment/web-app

# See details of a specific revision
kubectl rollout history deployment/web-app --revision=2

# 🔥 ROLLBACK to previous version
kubectl rollout undo deployment/web-app

# Rollback to a specific revision
kubectl rollout undo deployment/web-app --to-revision=1

# Scale up or down
kubectl scale deployment web-app --replicas=5

# Pause a rollout (freeze it mid-way)
kubectl rollout pause deployment/web-app

# Resume a paused rollout
kubectl rollout resume deployment/web-app
```

---

## 🔄 Rollout Strategies

### Strategy 1: RollingUpdate (Default) ✅
```
Before: [v1] [v1] [v1] [v1]
Step 1: [v2] [v1] [v1] [v1]
Step 2: [v2] [v2] [v1] [v1]
Step 3: [v2] [v2] [v2] [v1]
After:  [v2] [v2] [v2] [v2]  ← Zero downtime!
```
- `maxSurge`: How many extra pods can exist during update
- `maxUnavailable`: How many pods can be down during update

### Strategy 2: Recreate ⚠️
```
Before: [v1] [v1] [v1]
Step 1: [ ]  [ ]  [ ]   ← ALL OLD PODS DELETED (downtime!)
Step 2: [v2] [v2] [v2]  ← All new pods start
```
- Simpler, but **causes downtime**
- Only use when you NEED it (e.g., incompatible database schema changes)

---

## 🚨 Debugging Deployments

```bash
# Something's wrong? Start here:
kubectl get pods                          # See pod states
kubectl describe deployment web-app       # See events and conditions
kubectl describe pod <pod-name>           # See pod-level events
kubectl logs <pod-name>                   # See app logs
kubectl rollout status deployment/web-app # Is rollout stuck?
```

**Common issues:**
- 🔴 `ImagePullBackOff` → Wrong image name or no access to registry
- 🔴 `CrashLoopBackOff` → App crashes on startup → check logs!
- 🔴 Rollout stuck at 50% → readiness probe failing → pods not "ready"

---

## 🎤 Interview Knockout Answers

**Q: Why use a Deployment instead of a ReplicaSet?**
> "Deployments add rollout and rollback capabilities on top of ReplicaSets. While a ReplicaSet only maintains pod count, a Deployment also manages rolling updates, tracks revision history, and allows safe rollbacks. In production, we always use Deployments."

**Q: What is a rolling update?**
> "A rolling update gradually replaces old pods with new ones. The Deployment creates a new ReplicaSet and scales it up while scaling down the old ReplicaSet. This ensures zero downtime — some pods always remain available."

**Q: How does rollback work?**
> "Kubernetes keeps previous ReplicaSets. When you rollback, it scales the previous ReplicaSet back up and scales the current one to zero. You can target specific revisions with `--to-revision`."

**Q: What is maxSurge and maxUnavailable?**
> "maxSurge is the maximum number of extra pods that can be created above the desired count during a rolling update. maxUnavailable is the maximum number of pods that can be unavailable during the update. Together they control the speed and safety of the rollout."

---

## ⚡ 30-Second Revision

```
🚀 Deployment = ReplicaSet + Rolling Updates + Rollback
🔄 Rolling Update = replace pods gradually (zero downtime)
⏪ Rollback = go back to previous version instantly
📊 Creates a new ReplicaSet for each image update
⚙️  RollingUpdate (default) vs Recreate (with downtime)
🎮 kubectl rollout = your best friend for deploy ops
👑 Most used Kubernetes object in production!
```

---

> **Ravi, you're crushing it! 🔥** Deployment is the most important concept for your DevOps career. Revise this one multiple times! Next → [07-namespaces.md](07-namespaces.md) 🚀
