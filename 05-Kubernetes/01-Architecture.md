# ☸️ Kubernetes Architecture

## 🖼️ Quick Visual Summary

![Quick Summary: Kubernetes Architecture](../assets/topic-summaries/kubernetes-architecture.svg)

> **80/20 Summary:** Kubernetes turns many containers on many machines into one system you can control from YAML. 🧭

## 1. Big Picture

Ravi, focus on this first:

Kubernetes was created because running containers manually does not scale.
If you have one app, Docker is fine.
If you have many apps, many servers, failures, and releases, you need a control system.

Before Kubernetes, teams had to:

- start containers by hand
- restart failed containers manually
- manage networking themselves
- balance traffic themselves
- track which server was healthy

Kubernetes solved that by giving us a cluster manager that follows desired state.

## 2. Real-Life Analogy

Think of Kubernetes like an airport control system ✈️

- The **Control Plane** is the control tower
- The **Worker Nodes** are the runways and gates
- The **Pods** are the planes being handled
- The **Scheduler** decides where each plane should go
- The **Kubelet** on each node does the local work

Ravi, this analogy matters because Kubernetes is not just "run a container".
It is "manage the whole airport safely."

## 3. Technical Definition

Kubernetes is an open-source container orchestration platform that schedules, runs, scales, and self-heals containerized workloads across a cluster.

## 4. Internal Working

```text
Developer
   |
   | kubectl apply
   v
API Server
   |
   | writes desired state
   v
etcd
   |
   | scheduler picks a node
   v
Scheduler
   |
   | sends assignment to node
   v
Kubelet
   |
   | asks runtime to pull image
   v
Container Runtime
   |
   v
Pod Running
```

### Control Plane vs Worker Node

| Part | What it does |
| --- | --- |
| Control Plane | Makes decisions and stores cluster state 🧠 |
| Worker Node | Runs your containers 💪 |

### Main Components

| Component | Simple meaning |
| --- | --- |
| `kube-apiserver` | Entry point for all requests 🛂 |
| `etcd` | Stores the truth of the cluster 🗄️ |
| `kube-scheduler` | Chooses the right node 🎯 |
| `kube-controller-manager` | Fixes drift between desired and actual state 🔄 |
| `kubelet` | Runs on each node and manages Pods 🧰 |
| `kube-proxy` | Helps route traffic on the node 🌐 |
| Container runtime | Pulls and runs containers 📦 |

## 5. Key Concepts

- **Desired state:** what you want the cluster to look like
- **Actual state:** what is really running right now
- **Reconciliation:** Kubernetes keeps comparing the two and correcting drift
- **Cluster:** the full set of control plane + worker nodes
- **Namespace:** a logical grouping for resources
- **etcd:** the key-value store that holds cluster state

## 6. Commands

| Command | Why we use it | What it tells you |
| --- | --- | --- |
| `kubectl cluster-info` | Check if the cluster is reachable | Shows API server and cluster endpoints |
| `kubectl get nodes` | Check node health | Lists worker nodes and their status |
| `kubectl get pods -A` | See all Pods | Shows workloads across namespaces |
| `kubectl describe node <name>` | Debug one node | Shows capacity, conditions, and events |
| `kubectl get events -A --sort-by=.metadata.creationTimestamp` | Find recent issues | Shows the timeline of cluster activity |

## 7. Real Production Usage

Ravi, this is the version you will use in a real project:

- Most companies use EKS, GKE, or AKS.
- The cloud provider manages the control plane.
- Engineers manage namespaces, workloads, Services, RBAC, and observability.
- Platform teams watch node health, cluster alerts, and rollout status.

## 8. Common Mistakes

- ❌ Thinking Kubernetes is just "Docker with more steps"
  - Why it is wrong: Kubernetes solves cluster-wide scheduling and self-healing.
  - ✅ Correct: think in terms of desired state and controllers.

- ❌ Editing running containers manually
  - Why it is wrong: those changes are temporary.
  - ✅ Correct: change YAML and apply the new state.

- ❌ Ignoring `etcd`
  - Why it is wrong: the cluster loses its memory.
  - ✅ Correct: protect and back up control plane data.

## 9. Best Practices

1. Use managed Kubernetes unless you truly need self-managed control plane.
2. Treat nodes as disposable.
3. Use RBAC carefully.
4. Watch events, not just Pods.
5. Keep infrastructure declarative.

## 10. Interview Corner

Ravi, your interviewer might ask this:

**Q1: What is Kubernetes?**
A1: A container orchestration platform that manages workloads across a cluster.

**Q2: What is the Control Plane?**
A2: The part of Kubernetes that makes cluster decisions and stores state.

**Q3: What is `etcd`?**
A3: The cluster database that stores desired and current state.

**Q4: What does the `kubelet` do?**
A4: It runs on each node and makes sure the assigned Pods are running.

**Q5: Why is Kubernetes declarative?**
A5: You declare what you want, and controllers work to make reality match it.

## 11. Revision Summary

- Kubernetes manages containers at scale. 🚀
- Control Plane decides.
- Worker Nodes run workloads.
- `kubectl` talks to the API server.
- `etcd` stores cluster state.

## 12. Key Takeaways

- Kubernetes solves scaling and reliability.
- The control plane is the brain.
- The worker node is the execution layer.
- Desired state is the core idea.

## 13. Comparison Table

| Control Plane | Worker Node |
| --- | --- |
| Makes decisions | Runs Pods |
| Stores cluster state | Executes workloads |
| Hosts API, scheduler, controllers | Hosts kubelet and runtime |

## 14. Memory Tricks

- **Brain and muscle**: control plane thinks, worker node works
- **Desired vs actual**: Kubernetes keeps closing the gap
- **API, etcd, scheduler, controller**: the decision chain

## 15. Official Docs

- [Kubernetes Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
