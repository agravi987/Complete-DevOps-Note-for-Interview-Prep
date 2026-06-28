# 🧩 Kubernetes Components — Meet the Team!

> **Ravi! 👋** Architecture gave you the map. Now let's meet every player on the team, understand EXACTLY what their job is, and why they exist. After this, you'll be able to explain Kubernetes internals like a senior engineer! 😎

---

## 🍽️ The Restaurant Analogy (Extended!)

Last time we used a restaurant. Let's go deeper:

You walk into a restaurant and say **"I want pasta for 3 people."**

Here's what happens in Kubernetes terms:

```
You (kubectl)
  → tell the Waiter (API Server)
    → Waiter writes it in the Order Book (etcd)
      → Host assigns a table (Scheduler)
        → Manager checks everything's in order (Controller Manager)
          → Chef at your table gets to cooking (kubelet)
            → Food arrives on the table (Container Runtime → Pod)
              → The right guests get served (kube-proxy)
```

**Every component has ONE clear job. That's the beauty!** 🎯

---

## 🧠 Control Plane Components (The Brain)

### 1️⃣ API Server — The Front Door 🚪

```
kubectl → API Server → Everything else
```

- **What it does:** EVERY request (from you, from controllers, from the UI) goes through the API server first. It validates, authenticates, and then passes requests along.
- **Why it matters:** It's the ONLY way to talk to the cluster.
- **Interview line:** *"The API server is the single entry point for all cluster communication."*

---

### 2️⃣ etcd — The Brain's Memory 💾

```
"If etcd dies, the cluster forgets everything. 🪦"
```

- **What it does:** Stores ALL cluster state — pods, configs, secrets, deployments, literally everything
- **Technology:** Distributed key-value store (super reliable, uses Raft consensus)
- **Real talk:** In managed K8s (EKS, GKE, AKS), AWS/Google manages etcd. You don't touch it.
- **Interview line:** *"etcd is the source of truth. Backup etcd = backup your entire cluster."*

---

### 3️⃣ Scheduler — The Matchmaker 🎯

```
New Pod needs a home →  Scheduler finds the BEST node
```

- **What it checks:**
  - Node has enough CPU & RAM? ✅
  - Any affinity/anti-affinity rules? ✅
  - Node taint vs pod tolerations? ✅
- **After decision:** It updates etcd: "Pod X → Node 2"
- **Interview line:** *"The scheduler doesn't run pods — it just decides WHERE they run."*

---

### 4️⃣ Controller Manager — The Fixer 🔧

```
Desired State: 3 pods
Actual State:  2 pods (one crashed)
Controller:    "Creating 1 more pod NOW!" 💪
```

- **What it does:** Runs many controllers in one process. Each controller watches one type of resource.
- **Examples:**
  - 🔁 **ReplicaSet Controller** — keeps replica count correct
  - 🚀 **Deployment Controller** — manages rollouts
  - ⏰ **CronJob Controller** — runs jobs on schedule
- **Interview line:** *"Controllers are the reconciliation engines — they continuously compare desired vs actual state and fix any drift."*

---

## 💪 Worker Node Components (The Muscles)

### 5️⃣ kubelet — The Floor Manager 🤖

```
kubelet lives on EVERY worker node
kubelet = the agent that makes pods real
```

- **What it does:** Receives pod specs from API server → talks to container runtime → starts/stops containers → reports health back
- **Think of it as:** The local manager who takes orders from HQ and actually executes them
- **Interview line:** *"kubelet is the agent on each node. It ensures containers are running as specified."*

---

### 6️⃣ kube-proxy — The Traffic Cop 🚦

```
User request → kube-proxy → routes to correct Pod IP
```

- **What it does:** Manages network rules on each node. Makes Services work by routing traffic to the right pod endpoints.
- **Note:** It uses iptables or IPVS under the hood
- **Interview line:** *"kube-proxy implements Services networking — it makes sure when you hit a Service, traffic gets to the right Pod."*

---

### 7️⃣ Container Runtime — The Engine 🐳

```
kubelet says "run this image" → Container Runtime ACTUALLY does it
```

- **Examples:** `containerd` (most common), `CRI-O`, old-school `Docker`
- **Why not Docker directly?** Kubernetes uses CRI (Container Runtime Interface) — Docker was removed in v1.24!
- **Interview line:** *"The container runtime (usually containerd) is what actually pulls images and runs containers at the OS level."*

---

## 📊 Full Component Map

```
┌──────────────── CONTROL PLANE ────────────────┐
│                                               │
│   ┌──────────┐    ┌──────┐    ┌───────────┐  │
│   │API Server│    │ etcd │    │ Scheduler │  │
│   └────┬─────┘    └──────┘    └─────┬─────┘  │
│        │                            │         │
│   ┌────┴──────────────────────┐     │         │
│   │    Controller Manager     │◄────┘         │
│   └───────────────────────────┘               │
└───────────────────┬───────────────────────────┘
                    │
        ┌───────────┼──────────┐
        ↓           ↓          ↓
   ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ Node 1  │ │ Node 2  │ │ Node 3  │
   │kubelet  │ │kubelet  │ │kubelet  │
   │kube-prxy│ │kube-prxy│ │kube-prxy│
   │containrd│ │containrd│ │containrd│
   │ 🐳 Pod  │ │ 🐳 Pod  │ │ 🐳 Pods │
   └─────────┘ └─────────┘ └─────────┘
```

---

## ⚡ Quick Commands

```bash
# See system pods (in managed clusters these show control plane stuff)
kubectl get pods -n kube-system

# Check node health
kubectl get nodes

# Deep dive into a node
kubectl describe node <node-name>

# See what's running on a specific node
kubectl get pods -A --field-selector spec.nodeName=<node-name>
```

---

## 🎤 Interview Knockout Round

| Question | Perfect Answer |
|---|---|
| What does the API server do? | Single entry point, validates & routes all cluster requests |
| What is kubelet? | Node agent that runs pods as instructed by the API server |
| Why is etcd critical? | It stores ALL cluster state — it IS the cluster memory |
| What is the scheduler's job? | Assign pods to nodes based on resources & constraints |
| What is the controller manager? | Runs reconciliation loops to keep desired state = actual state |
| What is kube-proxy? | Implements Service networking by routing traffic to pod IPs |

---

## 🚨 Ravi, Watch Out For These Gotchas!

- ❌ **"kubelet is in the control plane"** — NO! kubelet is on WORKER nodes
- ❌ **"Docker is the container runtime"** — Kubernetes removed Docker support in v1.24. It's containerd now!
- ❌ **"The scheduler runs pods"** — No! Scheduler just DECIDES where to place them. kubelet RUNS them.

---

## ⚡ 30-Second Revision

```
Control Plane:
  🚪 API Server   → entry point for everything
  💾 etcd         → stores all cluster state
  🎯 Scheduler    → picks node for new pods
  🔧 Controller   → reconciles desired vs actual

Worker Node:
  🤖 kubelet      → runs pods on this node
  🚦 kube-proxy   → routes traffic to pods
  🐳 Runtime      → actually runs containers
```

---

> **Ravi, you now know every player on the team! 🏆** Understanding who does what is the foundation for debugging. When something breaks, you'll know EXACTLY which component to blame! Next → [03-pods.md](03-pods.md) 🚀
