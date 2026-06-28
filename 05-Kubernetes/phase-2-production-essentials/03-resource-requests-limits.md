# ⚖️ Resource Requests & Limits

> **Hey Ravi! 👋** Imagine a buffet where one guy eats all the food and leaves nothing for everyone else. 😤 That happens in Kubernetes if you don't set boundaries! **Requests & Limits** are how you tell Kubernetes exactly how much CPU and Memory a container is allowed to eat. 🍽️

---

## 🧠 What are Requests and Limits?

> **Requests** = The minimum guaranteed resources (Used for scheduling).
> **Limits** = The absolute maximum resources allowed (Used for enforcement).

---

## 🏨 The Hotel Reservation Analogy

- **Requests (Booking the Room):** I *request* 1 bed. The hotel (Scheduler) checks if there are any 1-bed rooms left. If yes, I get a room. If no, I wait outside (Pending state).
- **Limits (The Minibar Cap):** I have a *limit* of 2 sodas. I can drink 1 or 2. But if I try to drink a 3rd soda, Security (Kubelet) kicks me out (OOMKilled) or slows me down (CPU Throttled). 👮‍♂️

---

## 🔢 Understanding the Units

### 🧠 CPU (Measured in cores/millicores)
- CPU is **compressible**. If you hit the limit, you don't die — you just get slower (Throttled). 🐢
- `1` = 1 whole vCPU core.
- `500m` = 500 millicores (Half a core).
- *Minimum possible is usually `1m`.*

### 🐏 Memory (Measured in Bytes)
- Memory is **incompressible**. If you try to use more memory than your limit, the Linux kernel kills your process instantly! 💀 (OOMKilled).
- `256Mi` = 256 Mebibytes (Standard binary MB).
- `1Gi` = 1 Gibibyte (~1 GB).

---

## 📄 Full YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: my-app
    image: nginx:1.27
    resources:
      requests:              # SCHEDULING GUARANTEE
        cpu: "250m"          # 1/4 of a core
        memory: "128Mi"      # 128 MB RAM
      limits:                # ENFORCEMENT CAP
        cpu: "500m"          # Max 1/2 core (will be throttled if exceeded)
        memory: "256Mi"      # Max 256 MB (will be OOMKilled if exceeded)
```

---

## 🏆 QoS Classes (Quality of Service)

Kubernetes quietly assigns a "VIP status" to your pod based on how you set these numbers. If a node runs out of memory, K8s evicts pods in this order (lowest VIP first):

| Class | How to get it | Eviction Priority |
|---|---|---|
| **BestEffort** | No requests or limits set at all. | 🥉 1st to be killed! (Expendable) |
| **Burstable** | Requests are lower than Limits. | 🥈 2nd to be killed. |
| **Guaranteed** | Requests EXACTLY equal Limits. | 🥇 VIP! Last to be killed. |

> 💡 **Ravi's Tip:** For critical databases or production apps, ALWAYS set `requests == limits` to get Guaranteed QoS!

---

## 🛡️ Guardrails: ResourceQuota & LimitRange

If you share a cluster, developers will request 100 CPUs just because they can. 🤦‍♂️ Admin tools to the rescue!

### 1. ResourceQuota (Namespace Level)
"This entire `dev` namespace can only use a TOTAL of 10 CPUs and 20Gi Memory combined across all pods."

### 2. LimitRange (Pod Level)
"Every single container in this namespace MUST have at least 100m CPU, but CANNOT exceed 1 CPU. Also, if they forget to set limits, I will automatically inject a default of 250m CPU."

---

## 🎮 Commands & Debugging

```bash
# See how much CPU/Memory nodes are using
kubectl top nodes

# See how much CPU/Memory pods are using (requires metrics-server!)
kubectl top pods

# See why a pod is failing
kubectl describe pod my-app
# Look for: "State: Terminated, Reason: OOMKilled" 💀

# See if a pod is pending due to lack of space
kubectl describe pod my-pending-pod
# Look for: "FailedScheduling: 0/3 nodes are available: 3 Insufficient memory"
```

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between a resource Request and a Limit?**
> "Requests are used by the Kubernetes Scheduler to find a node with enough capacity for the pod; it's a guaranteed baseline. Limits are enforced by the Kubelet and container runtime; they act as a hard cap. If a container exceeds its CPU limit, it gets throttled. If it exceeds its memory limit, it gets OOMKilled."

**Q: What is OOMKilled and how do you fix it?**
> "OOMKilled means Out Of Memory Killed. It happens when a container tries to consume more RAM than its defined `limit`. To fix it, you either need to increase the memory limit in the YAML, or fix the memory leak in the application code."

**Q: Explain QoS classes in Kubernetes.**
> "Quality of Service classes determine which pods get evicted first when a node faces resource starvation. 'BestEffort' (no requests/limits) gets evicted first. 'Burstable' (requests < limits) gets evicted next. 'Guaranteed' (requests == limits) is evicted last."

---

## ⚡ 30-Second Revision

- 🙋‍♂️ **Requests** = "I need this to start." (Scheduling).
- 🛑 **Limits** = "Don't let me use more than this." (Enforcement).
- 🐢 Exceed CPU Limit = App gets slow (Throttled).
- 💀 Exceed Memory Limit = App crashes (`OOMKilled`).
- 🥇 **Guaranteed QoS** = Best for prod (Requests == Limits).
- 🛡️ **ResourceQuota** = Total limit for a Namespace.
- 🛡️ **LimitRange** = Default/Max limits for individual Pods.

> Next → [Persistent Volumes](04-persistent-volumes.md) 🚀
