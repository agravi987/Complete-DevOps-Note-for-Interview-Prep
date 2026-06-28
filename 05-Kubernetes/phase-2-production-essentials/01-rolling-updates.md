# 🔄 Rolling Updates — Smooth Sailing for Upgrades

> **Hey Ravi! 👋** Upgrading an app used to mean sending an email: "System down from 2 AM to 4 AM for maintenance." Gross, right? 🤮 With Kubernetes **Rolling Updates**, you can upgrade your app in the middle of the day, and your users won't even notice. Let's see how this magic works! ✨

---

## 🧠 What is a Rolling Update?

> **A Rolling Update gradually replaces old Pods with new Pods, ensuring that at least some Pods are always available to handle traffic.**

The Deployment creates a **NEW** ReplicaSet and scales it up step-by-step, while simultaneously scaling down the **OLD** ReplicaSet. 

---

## 🛗 The Elevator Analogy

Imagine a busy 4-elevator bank in a hotel. You need to upgrade the elevators.

- **Bad Way (Recreate):** Shut down ALL 4 elevators at once. Upgrade them. Turn them all back on. Guests take the stairs for hours. 🏃‍♂️💨
- **Good Way (Rolling Update):** Shut down Elevator 1. Upgrade it. Turn it back on. Then do Elevator 2, then 3, then 4. There are ALWAYS 3 elevators running! 🛗✅

---

## 🏗️ How it Works Internally

```text
Desired Replicas: 3
Updating from v1 to v2...

Step 1: 
  ReplicaSet v1: [v1] [v1] [v1]  (3 running)
  ReplicaSet v2: [  ]            (0 running)

Step 2 (Surge): 
  ReplicaSet v1: [v1] [v1] [v1]  
  ReplicaSet v2: [v2]            (1 new pod starts)

Step 3 (Scale down): 
  ReplicaSet v1: [v1] [v1]       (1 old pod terminates)
  ReplicaSet v2: [v2] [v2]       (1 more new pod starts)

Step 4 (Done!): 
  ReplicaSet v1: [  ]            (Scaled to 0, but KEPT for rollback)
  ReplicaSet v2: [v2] [v2] [v2]  (All running!)
```

---

## 📄 YAML Manifest Explained

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-demo
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%          # 1 extra pod allowed during update
      maxUnavailable: 25%    # 1 pod allowed to be down during update
  selector:
    matchLabels:
      app: rolling-demo
  template:
    metadata:
      labels:
        app: rolling-demo
    spec:
      containers:
      - name: app
        image: nginx:1.27    # Change this to trigger rollout!
        readinessProbe:      # 🚨 CRITICAL FOR ROLLOUTS!
          httpGet:
            path: /
            port: 80
```

### 🎛️ maxSurge & maxUnavailable
- **maxSurge**: How many extra Pods above `replicas` can be created during the update? (Can be absolute number like `1` or percentage like `25%`).
- **maxUnavailable**: How many Pods below `replicas` are allowed to be unavailable during the update?

---

## 🎮 Rollout Commands (Your Best Friends)

```bash
# Trigger an update (change the image)
kubectl set image deployment/rolling-demo app=nginx:1.28

# Watch the progress live! 🍿
kubectl rollout status deployment/rolling-demo

# See the history of deployments
kubectl rollout history deployment/rolling-demo

# PANIC BUTTON: The new version is broken! Undo! ⏪
kubectl rollout undo deployment/rolling-demo

# Roll back to a specific revision
kubectl rollout undo deployment/rolling-demo --to-revision=2

# Pause the rollout (good for canary testing)
kubectl rollout pause deployment/rolling-demo

# Resume it after checking
kubectl rollout resume deployment/rolling-demo
```

---

## 🆚 RollingUpdate vs Recreate

| Feature | RollingUpdate (Default) | Recreate |
|---|---|---|
| **Downtime?** | ❌ No | ✅ Yes (All old pods killed first) |
| **Speed** | Slower (step-by-step) | Faster (kill all, start all) |
| **Resource Usage** | Needs extra nodes/CPU for surge | Low (no overlap) |
| **When to use?** | Web apps, APIs, stateless | DB schema changes, single PVCs |

---

## 🎤 Interview Knockout Answers

**Q: What is a rolling update in Kubernetes?**
> "A rolling update is a deployment strategy that replaces old pods with new ones gradually, ensuring zero downtime. The Deployment controller creates a new ReplicaSet and scales it up while scaling down the old one, controlled by `maxSurge` and `maxUnavailable`."

**Q: Why are Readiness Probes critical during a rolling update?**
> "Without a readiness probe, Kubernetes assumes a pod is ready as soon as the container process starts. It might kill old pods before the new ones can actually serve traffic, causing downtime! Readiness probes tell K8s exactly when it's safe to scale down the old pods."

**Q: What does maxSurge and maxUnavailable do?**
> "`maxSurge` controls how many extra pods can be created above the desired replica count during an update (speeding up rollout). `maxUnavailable` controls how many pods can be unavailable below the desired count (controlling capacity reduction)."

---

## ⚡ 30-Second Revision

- 🔄 Rolling updates = zero downtime upgrades.
- 🏗️ Deployment manages it by juggling two ReplicaSets.
- 📈 `maxSurge` = extra pods allowed.
- 📉 `maxUnavailable` = missing pods allowed.
- ⏪ `kubectl rollout undo` is your panic button.
- 🚨 ALWAYS use Readiness Probes with rolling updates!
- ⚠️ Recreate strategy causes downtime, use only when necessary.

> Next → [Liveness & Readiness Probes](02-liveness-readiness-probes.md) 🚀
