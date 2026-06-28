# 🛡️ Pod Disruption Budgets (PDB)

> **Hey Ravi! 👋** Imagine you run a highly available Database cluster with 3 replicas. It needs a strict quorum (majority) of 2 to survive. Suddenly, the Kubernetes Admin decides to upgrade the cluster and reboots all the nodes at once. Your DB goes down! 💥 How do you, the developer, tell the admin "DO NOT touch my pods if it drops me below 2 replicas"? You use a **Pod Disruption Budget**! 🛡️

---

## 🧠 What is a Pod Disruption Budget (PDB)?

> **A PDB is a rule that limits the number of Pods of a replicated application that are allowed to be down simultaneously due to VOLUNTARY disruptions.**

---

## 🚨 Voluntary vs Involuntary Disruptions

A PDB can **only** protect you from things K8s controls (Voluntary). It cannot protect you from an asteroid hitting the data center! ☄️

| Voluntary (PDB Protects You! 🛡️) | Involuntary (PDB Can't Help 🤷‍♂️) |
|---|---|
| Admin running `kubectl drain node` | AWS underlying hardware failure |
| Cluster Autoscaler deleting a node | Node running out of memory (OOM) |
| Upgrading the Kubernetes version | Kernel panic on the Linux host |
| Deleting a pod manually | Someone physically unplugs the server |

---

## 📄 Real YAML (minAvailable vs maxUnavailable)

You can define a PDB using either an absolute number or a percentage.

### Option 1: Keep at least X alive (`minAvailable`)
*"I have 3 instances of ZooKeeper. I absolutely MUST have 2 running at all times to maintain quorum."*

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: zookeeper-pdb
spec:
  minAvailable: 2           # OR "66%"
  selector:
    matchLabels:
      app: zookeeper
```

### Option 2: Allow at most X to be down (`maxUnavailable`)
*"I have 10 web servers. I don't care how many I have total, just don't take down more than 2 at the exact same time during node upgrades!"*

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  maxUnavailable: 2         # OR "20%"
  selector:
    matchLabels:
      app: web-server
```

---

## 🛑 How PDB Blocks the Admin (`kubectl drain`)

When a K8s Admin needs to upgrade a Node, they run `kubectl drain <node>`. This safely evicts all pods off the node.

But! If evicting a pod violates the PDB (e.g., it would drop ZooKeeper to 1 replica), the `kubectl drain` command will literally **BLOCK and wait**. ✋ 

It waits until Kubernetes spins up a replacement pod on another node. Once the replica count is back to a safe level, it allows the eviction to proceed.

---

## ⚠️ The Danger: Deadlocks!

If you set a PDB of `minAvailable: 100%`, you are basically saying "My application is never allowed to have a single pod evicted."
If the Cluster Autoscaler tries to scale down an empty node, it will be BLOCKED forever because it's not allowed to evict your pod! 

> **Rule of thumb:** Never set `minAvailable: 100%` or `maxUnavailable: 0` unless you know exactly what you are doing. It breaks cluster automation!

---

## 🎤 Interview Knockout Answers

**Q: What is a Pod Disruption Budget and why is it important for highly available applications?**
> "A PDB is a policy that limits how many pods of an application can be brought down simultaneously during voluntary disruptions, like node maintenance or cluster autoscaling. It is critical for stateful apps like databases or consensus systems (like etcd/ZooKeeper) that require a strict quorum to avoid data corruption or total downtime."

**Q: What is the difference between a voluntary and involuntary disruption?**
> "Involuntary disruptions are unpreventable hardware or OS failures, like a VM crashing or a network outage. PDBs cannot protect against these. Voluntary disruptions are intentional actions initiated by cluster administrators or automation, like running `kubectl drain` to upgrade a node. PDBs intercept and delay these voluntary actions to ensure application availability."

**Q: How does a PDB interact with the Cluster Autoscaler?**
> "When the Cluster Autoscaler determines a node is underutilized and wants to terminate it to save money, it must first evict the pods on that node. If evicting those pods violates their configured PDB, the Cluster Autoscaler will abort the scale-down event, leaving the node running."

---

## ⚡ 30-Second Revision

- 🛡️ **PDB** = Protects apps from cluster maintenance/downtime.
- 🏗️ Only protects against **Voluntary** disruptions (like `kubectl drain`).
- 📈 `minAvailable` = Minimum pods that MUST be running.
- 📉 `maxUnavailable` = Maximum pods allowed to be offline.
- 🚦 Blocks node upgrades and Cluster Autoscaler scale-downs if violated.
- ⚠️ Never set `maxUnavailable: 0` (it causes deadlocks!).

---

> **Phase 3 Complete! 🎉** You now know how to secure a cluster, route traffic securely, and build elastic, self-healing infrastructure. Next stop: Advanced Platform Engineering!
> **Back to:** [Phase 3 README](README.md) | **Next Phase:** [Phase 4 - Advanced](../phase-4-advanced/README.md) 🚀
