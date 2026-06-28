# 👾 DaemonSets — One Per Node, Guaranteed!

> **Hey Ravi! 👋** Imagine you run a hotel. You don't care which room a guest sleeps in (Deployments). BUT, you absolutely MUST put exactly ONE smoke detector in EVERY single room! You can't have 2 in one room and 0 in another. In Kubernetes, when you need exactly **one pod on every node**, you use a **DaemonSet**! 🚨

---

## 🧠 What is a DaemonSet?

> **A DaemonSet ensures that a copy of a specific Pod runs on ALL (or some selected) Nodes in the cluster.**

When a new Node is added to the cluster, the DaemonSet automatically adds the Pod to it. When a Node is removed, the Pod is garbage collected.

---

## 🧹 The Janitor Analogy

- **Deployment:** "I need 5 janitors working right now. I don't care if they are all on Floor 1, as long as 5 are working."
- **DaemonSet:** "I need exactly 1 janitor assigned to EVERY floor in the building. If we build a new floor, immediately send a janitor there." 🧽

---

## 🎯 Common Use Cases (Why do we need this?)

You almost NEVER run your web app as a DaemonSet. You use it for infrastructure/platform agents:

1. **Logging Agents:** `fluentd` or `logstash` to collect logs from every node.
2. **Monitoring Agents:** `Prometheus Node Exporter` or `Datadog agent` to track CPU/Memory of every node.
3. **Networking (CNI):** `Calico` or `Weave` routing pods run as DaemonSets to manage network rules on every machine!
4. **Security:** Anti-virus or security scanners.

---

## 📄 Real YAML Example

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-logger
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14
        
        # Usually need to mount the host's log folders!
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          
      # Let the Pod read the Node's actual hard drive
      volumes:
      - name: varlog
        hostPath:
          path: /var/log

      # Optional: Only run on nodes with a specific label!
      nodeSelector:
        diskType: ssd
```

---

## 🎛️ Advanced: Running on Control Plane Nodes

By default, DaemonSets only run on Worker Nodes. Control Plane (Master) nodes have a "Taint" on them that repels pods. 

If you WANT your logging agent to run on the Master nodes too (to collect API server logs), you have to add a **Toleration** to your DaemonSet:

```yaml
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
        operator: Exists
```
*(We'll dive deeper into Taints and Tolerations in Phase 3!)*

---

## 🎮 Commands

```bash
# Get DaemonSets
kubectl get daemonset
kubectl get ds       # Short form!

# See the pods it created (You'll see one for every node)
kubectl get pods -o wide | grep fluentd

# Update a DaemonSet
kubectl set image ds/fluentd-logger fluentd=fluentd:v1.15
# (It performs a rolling update across nodes, one by one!)
```

---

## 🎤 Interview Knockout Answers

**Q: What is the main difference between a Deployment and a DaemonSet?**
> "A Deployment ensures a specific number of replicas are running across the cluster, regardless of which node they land on. A DaemonSet guarantees that exactly one copy of a Pod runs on every single node (or a subset of nodes based on nodeSelectors)."

**Q: Give me 3 use cases for a DaemonSet.**
> "DaemonSets are used for node-level infrastructure services. Examples include: 1. Cluster logging agents like FluentBit. 2. Node monitoring agents like Prometheus Node Exporter. 3. Container Network Interface (CNI) plugins like Calico or Flannel."

**Q: How can you make a DaemonSet run on only a specific set of nodes?**
> "You can use `nodeSelector` or `nodeAffinity` in the Pod spec of the DaemonSet. For example, you might only want a GPU-monitoring DaemonSet to run on nodes labeled with `hardware=gpu`."

---

## ⚡ 30-Second Revision

- 👾 **DaemonSet** = Exactly ONE pod on EVERY node.
- 📉 New node joins? Pod is added. Node dies? Pod is removed.
- 🎯 Use for: Logs (Fluentd), Monitoring (Prometheus), Network (Calico).
- 🏷️ Use `nodeSelector` if you only want it on *some* nodes.
- 🛡️ Use `tolerations` if you want it on Control Plane nodes.

> Next → [Ingress](08-ingress.md) 🚀
