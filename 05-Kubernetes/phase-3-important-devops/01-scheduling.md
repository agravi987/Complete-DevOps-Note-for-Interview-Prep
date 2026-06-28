# 📅 Scheduling (Affinity, Taints, Tolerations)

> **Hey Ravi! 👋** When you create a Pod, how does Kubernetes decide which Node it goes to? Does it just guess? Pick randomly? Nope! The **kube-scheduler** is incredibly smart. And sometimes, you need to tell it exactly where you want your Pods to go (like "Keep databases away from web servers!"). Let's learn how to boss the scheduler around! 🗣️

---

## 🧠 How the Scheduler Works

The `kube-scheduler` makes decisions in 2 phases:
1. **Filtering (Predicates):** Eliminates nodes that CANNOT run the pod (e.g., node is out of memory, or ports are taken).
2. **Scoring (Priorities):** Ranks the remaining nodes (e.g., this node has the most free space, give it a score of 10). The node with the highest score wins! 🏆

---

## 🧲 1. NodeSelector (The Simple Magnet)

The easiest way to put a pod on a specific node. Just match a label!

```yaml
spec:
  nodeSelector:
    diskType: ssd     # Only schedule on nodes with label: diskType=ssd
```

---

## 💍 2. Node Affinity (The Smart Magnet)

NodeSelector is too simple. What if I want `diskType=ssd` OR `diskType=nvme`? What if I prefer SSDs, but I'll settle for HDD if I have to?

```yaml
spec:
  affinity:
    nodeAffinity:
      # REQUIRED: Must have this! (Hard rule)
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: diskType
            operator: In
            values: ["ssd", "nvme"]
      
      # PREFERRED: Would be nice! (Soft rule - adds "Score" points)
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values: ["us-east-1a"]
```

---

## 👯 3. Pod Affinity & Anti-Affinity

Sometimes it's not about the Node, it's about the OTHER PODS.

- **PodAffinity:** "I want to be scheduled ON THE SAME NODE as my database pod so network latency is zero!" (The Clingy Friend).
- **PodAntiAffinity:** "I want to be scheduled ON A DIFFERENT NODE than my duplicate web server so if the node dies, we don't both die!" (Social Distancing).

```yaml
spec:
  affinity:
    podAntiAffinity:   # SOCIAL DISTANCING!
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: web-server
        topologyKey: "kubernetes.io/hostname"  # "Must not be on the same hostname"
```

---

## 🕷️ 4. Taints and Tolerations (The Bug Spray)

Affinity is a Pod saying "I want to go there!"
**Taints** are a Node saying "Stay away from me!" 

Imagine spraying Bug Spray (Taint) on a Node. All Pods stay away. ONLY Pods that carry the Antidote (Toleration) are allowed to land there!

**Step 1: Taint the Node (Admin does this)**
```bash
# Add a taint: key=gpu, value=true, effect=NoSchedule
kubectl taint nodes node1 gpu=true:NoSchedule
```

**Step 2: Add Toleration to Pod (Dev does this)**
```yaml
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

### The 3 Taint Effects:
1. `NoSchedule`: New pods won't be scheduled here. Existing pods are fine.
2. `PreferNoSchedule`: Try not to schedule here, but if the cluster is full, okay.
3. `NoExecute`: EVIL! 😈 Evicts (kills) any existing pods on the node that don't have the toleration immediately!

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between NodeAffinity and Taints/Tolerations?**
> "NodeAffinity is a property of Pods that attracts them to a set of Nodes (either as a hard requirement or soft preference). Taints, on the other hand, are applied to Nodes to repel Pods. A Node with a taint will not accept any Pods unless the Pod explicitly has a matching Toleration. NodeAffinity attracts; Taints repel."

**Q: Can you explain the 'topologyKey' in PodAntiAffinity?**
> "The `topologyKey` defines the domain for the anti-affinity rule. If you set it to `kubernetes.io/hostname`, the scheduler ensures the pods don't land on the same exact machine. If you set it to `topology.kubernetes.io/zone`, the scheduler ensures the pods don't land in the same cloud Availability Zone, giving you high availability across data centers."

**Q: What does the `NoExecute` taint effect do?**
> "`NoExecute` is a strict taint effect. Not only does it prevent new pods without the toleration from being scheduled on the node, but it also actively evicts (kills) any pods already running on the node that do not have the matching toleration."

---

## ⚡ 30-Second Revision

- 🧠 **kube-scheduler** = Filters, then Scores nodes.
- 🧲 **NodeSelector** = Simple exact match for labels.
- 💍 **NodeAffinity** = Smart matching (Required vs Preferred).
- 👯 **PodAffinity** = "Put me next to this pod."
- 🙅 **PodAntiAffinity** = "Keep me away from this pod." (For High Availability!)
- 🕷️ **Taints (Node)** = "Stay away unless you have the antidote."
- 💊 **Tolerations (Pod)** = The antidote.

> Next → [Role-Based Access Control (RBAC)](02-rbac.md) 🚀
