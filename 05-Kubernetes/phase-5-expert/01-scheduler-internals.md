# 🧠 Scheduler Internals

> **Hey Ravi! 👋** We learned how to use NodeAffinity and Taints to tell the Scheduler what to do. But how does the Scheduler *actually* think? How does it look at 5,000 nodes and pick the absolute perfect one in less than a second? Let's crack open the Scheduler's brain and see how it ticks! ⏱️

---

## 🧠 The Scheduling Cycle

The `kube-scheduler` doesn't just guess. It runs a highly optimized, two-step pipeline for every single Pod in the `Pending` queue.

### Step 1: Filtering (Predicates) 🚫
The scheduler takes all 5,000 nodes and asks hard "Yes/No" questions.
- Does this node have enough CPU? (No? Drop it.)
- Does this node have the matching NodeAffinity label? (No? Drop it.)
- Does this node have a Taint we can't tolerate? (Yes? Drop it.)
*Result: We are left with 50 nodes that CAN run the pod.*

### Step 2: Scoring (Priorities) 🏆
The scheduler takes the remaining 50 nodes and scores them from 0 to 100 based on soft preferences.
- Does putting the pod here balance the overall cluster CPU? (+20 points)
- Does it meet a "preferred" NodeAffinity? (+10 points)
- Does it already have the Docker image downloaded? (+5 points)
*Result: The node with the highest score wins!*

---

##  صف The Scheduler Queues

When a Pod is created, it doesn't just go to the node instantly. It goes through queues:

1. **activeQ:** Pods ready to be scheduled right now.
2. **backoffQ:** Pods that failed scheduling recently (e.g., no room). The scheduler waits a bit before trying them again (exponential backoff).
3. **unschedulableQ:** Pods that are impossible to schedule (e.g., asked for a node label that doesn't exist anywhere). They sit here until an event happens (like a new node joining) that moves them back to the activeQ.

---

## 🥊 Priority and Preemption (The VIP Pass)

What happens if the cluster is 100% full, but you need to deploy a critical production hotfix? It will be stuck in `Pending`! 😱

Enter **PriorityClasses**. You can assign a PriorityClass to your critical pods. 

If a High-Priority pod is `Pending` because the cluster is full, the scheduler will trigger **Preemption**. It will ruthlessly murder (evict) low-priority pods (like batch processing jobs) to make room for the VIP pod! 🩸

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-critical
value: 1000000          # The higher the number, the more important!
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```

---

## 👯 Gang Scheduling (For ML & Big Data)

By default, Kubernetes schedules Pods one-by-one. 
But what if you have a Machine Learning job that requires exactly 10 Pods to run simultaneously, or else it fails? If K8s schedules 8 pods and runs out of room, those 8 pods sit there doing nothing, wasting resources!

**Gang Scheduling** (using custom schedulers like Volcano or Kube-batch) ensures "All or Nothing" scheduling. It waits until there is room for ALL 10 pods, and schedules them simultaneously! 🧠

---

## 🎤 Interview Knockout Answers

**Q: Explain the two main phases of the kube-scheduler.**
> "The scheduler operates in two main phases: Filtering (also known as Predicates) and Scoring (Priorities). Filtering evaluates hard constraints—like CPU requests, taints, and required node affinities—to eliminate nodes that cannot run the pod. Scoring evaluates soft constraints—like image locality or preferred affinities—to rank the remaining eligible nodes. The node with the highest score is selected."

**Q: What is Preemption in Kubernetes scheduling?**
> "Preemption occurs when the cluster is full, and a Pod with a high PriorityClass is stuck in a Pending state. The scheduler will identify lower-priority Pods currently running on nodes and evict (kill) them to free up enough resources to schedule the high-priority Pod."

**Q: Can you run multiple schedulers in the same Kubernetes cluster?**
> "Yes! You can run custom schedulers alongside the default `kube-scheduler`. When defining a Pod in its YAML, you can specify `spec.schedulerName`. If left blank, the default scheduler handles it. If specified, the default scheduler ignores the Pod, and your custom scheduler takes responsibility for placing it."

---

## ⚡ 30-Second Revision

- 🔍 **Filtering (Predicates)** = Hard rules (Yes/No). Eliminates bad nodes.
- 🏆 **Scoring (Priorities)** = Soft rules (0-100). Picks the best node.
-  صف **activeQ, backoffQ, unschedulableQ** = The internal waiting rooms for Pods.
- 🥊 **PriorityClass** = Gives pods VIP status.
- 🩸 **Preemption** = Killing low-priority pods to make room for VIP pods.
- 👯 **Gang Scheduling** = "All or Nothing" scheduling for AI/ML workloads.

> Next → [Controller Internals](02-controller-internals.md) 🚀
