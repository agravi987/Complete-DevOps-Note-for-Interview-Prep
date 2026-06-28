# 📊 Metrics Server & Monitoring

> **Hey Ravi! 👋** If a tree falls in a forest and no one is around, does it make a sound? If a Pod runs out of CPU and K8s isn't watching, how does it know to auto-scale? 🤔 It doesn't! Kubernetes is blind to CPU and Memory usage by default. We have to give it eyes using the **Metrics Server**. 👀

---

## 🧠 What is the Metrics Server?

> **The Metrics Server is a lightweight aggregator that collects CPU and Memory usage data from every node (via the kubelet) and exposes it to the Kubernetes API.**

Without the Metrics Server:
- `kubectl top pods` will return an error! ❌
- **Horizontal Pod Autoscaler (HPA)** will not work! ❌

---

## 🌡️ The Thermometer Analogy

- **Kubelet (The Nurse):** Checks the temperature (CPU/Memory) of the patient (Pod) every 15 seconds.
- **Metrics Server (The Clipboard):** The nurse writes the temps down on a clipboard.
- **HPA (The Doctor):** Looks at the clipboard and says "Temp is too high! Administer medicine! (Scale up!)" 👨‍⚕️📈

---

## ⚠️ Metrics Server vs Prometheus

This is a huge point of confusion. Let's clear it up!

| Feature | Metrics Server 📉 | Prometheus + Grafana 📊 |
|---|---|---|
| **What is it?** | A tiny pipeline for immediate data. | A massive database for historical data. |
| **Stores history?** | ❌ NO. Only knows "right now". | ✅ YES. Can query data from 6 months ago. |
| **What uses it?** | HPA, `kubectl top`. | Humans (dashboards), Alerts (PagerDuty). |
| **Complexity** | Extremely simple (stateless). | Very complex (stateful). |

> **Ravi's Rule:** Metrics Server is for the **cluster's internal brain** to make quick scaling decisions. Prometheus is for **human engineers** to debug, monitor, and create beautiful charts. You usually need BOTH in production!

---

## 🎮 Commands

```bash
# How to check if Metrics Server is installed and working!
kubectl get apiservices | grep metrics

# See which nodes are sweating 🥵
kubectl top nodes
# Output:
# NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
# worker-1   500m         25%    2048Mi          50%

# See which pods are eating all the resources!
kubectl top pods
kubectl top pods -n my-namespace

# Sort pods by CPU usage (Find the villain!)
kubectl top pods --sort-by=cpu
```

---

## 🛠️ How it works under the hood

1. The container runtime (containerd/Docker) calculates cgroup CPU/Mem usage.
2. The `kubelet` (running on every node) gathers this data.
3. The `metrics-server` pod reaches out to every `kubelet` and pulls the data into a central cache in memory.
4. It exposes this data at a specific API endpoint: `metrics.k8s.io`.
5. `kubectl` or `HPA` queries that endpoint!

---

## 🎤 Interview Knockout Answers

**Q: Why do we need to install the Metrics Server, and what breaks if it's missing?**
> "Kubernetes does not have a built-in mechanism to expose CPU and Memory usage to the control plane. The Metrics Server bridges this gap by aggregating resource usage data from the kubelets. If it's missing, commands like `kubectl top` will fail, and more importantly, the Horizontal Pod Autoscaler (HPA) will not be able to scale workloads because it cannot read the current metrics."

**Q: Can I use the Metrics Server to see how much CPU my pod used yesterday?**
> "No, the Metrics Server is strictly an in-memory aggregator for near-real-time data (usually the last ~15-60 seconds). It is not a time-series database. To view historical metrics from yesterday, you would need a full monitoring stack like Prometheus."

**Q: How does the Metrics Server actually get the data?**
> "The Metrics Server periodically scrapes the Summary API exposed by the `kubelet` on every worker node in the cluster. The `kubelet` gets that data directly from the container runtime (like containerd) via cgroups."

---

## ⚡ 30-Second Revision

- 👀 **Metrics Server** gives K8s the ability to "see" CPU/Memory usage.
- ⚙️ It powers `kubectl top` and the **HPA (Autoscaler)**.
- 🕰️ It has **NO MEMORY** of the past. It only knows right now.
- 📊 For historical charts and alerts, use **Prometheus + Grafana**.
- 🛠️ It pulls data directly from the `kubelet` on every node.

> Next → [Cluster Autoscaler](07-cluster-autoscaler.md) 🚀
