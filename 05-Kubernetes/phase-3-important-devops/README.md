# 🛠️ Phase 3 - Important for DevOps

> **Hey Ravi! 👋** You've mastered how to deploy apps (Phase 1) and keep them running smoothly (Phase 2). Now it's time to put on your **DevOps/Platform Engineer** hat. 🎩 Phase 3 is all about controlling the environment: securing permissions, isolating network traffic, and building massive, elastic, auto-scaling infrastructure. Let's build a real platform! 🏗️

---

## 🎯 What You'll Be Able to Do After Phase 3

By the end of these 8 topics, you will:
- ✅ Tell the scheduler EXACTLY which physical nodes your pods should run on
- ✅ Secure your cluster so developers can't accidentally delete production
- ✅ Block pods from talking to each other using internal firewalls
- ✅ Package complex apps and install them with one command
- ✅ Auto-scale your underlying cloud servers to save money
- ✅ Prevent administrators from breaking your highly available databases during maintenance

---

## 📚 The 8 DevOps Topics

| # | Topic | What You'll Learn | Vibe |
|---|---|---|---|
| 1 | [📅 Scheduling](01-scheduling.md) | Affinity, Taints, & Tolerations | 🧲 The Magnets |
| 2 | [🔐 RBAC](02-rbac.md) | Roles & RoleBindings (Permissions) | 🪪 VIP Club |
| 3 | [🤖 Service Accounts](03-service-accounts.md) | Identity and AWS/GCP permissions for Pods | 💳 Robot ID Cards |
| 4 | [🧱 Network Policies](04-network-policies.md) | Internal firewalls (Ingress/Egress) | 🏰 Castle Walls |
| 5 | [📦 Helm](05-helm.md) | The Kubernetes Package Manager | 🛍️ Ikea Furniture |
| 6 | [📊 Metrics Server](06-metrics-server-monitoring.md) | Giving K8s eyes for CPU/Memory | 👀 The Thermometer |
| 7 | [🏗️ Cluster Autoscaler](07-cluster-autoscaler.md) | Auto-buying servers to fit pending pods | 🚌 Bus Company |
| 8 | [🛡️ Pod Disruption Budgets](08-pod-disruption-budgets.md) | Protecting apps from admin maintenance | 🚦 The Stop Sign |

---

## 💡 Study Tips for the DevOps Mindset

> **1. Think "Least Privilege"**
> When studying RBAC and Network Policies, always default to "No". Deny all traffic. Deny all permissions. Then, explicitly write YAML to allow exactly what is needed, and nothing more.

> **2. The Autoscaling Trinity**
> Understand how the 3 scaling tools interact: 
> - **Metrics Server** (Sees the CPU spike) → **HPA** (Creates new Pods) → **Cluster Autoscaler** (Creates new Nodes for those Pods). They are a team! 🤝

> **3. Interview Focus**
> Interviewers LOVE Taints vs Affinity, and RBAC scenarios. Make sure you can explain the difference between a Role (namespace) and a ClusterRole (global).

---

## 🏁 Progress Tracker

Track your progress here (edit this file!):

- [ ] 01 - Scheduling (Affinity/Taints) ⭐ (Heavy interview topic)
- [ ] 02 - RBAC (Role-Based Access Control)
- [ ] 03 - Service Accounts
- [ ] 04 - Network Policies ⭐
- [ ] 05 - Helm
- [ ] 06 - Metrics Server & Monitoring
- [ ] 07 - Cluster Autoscaler
- [ ] 08 - Pod Disruption Budgets

---

> **Let's build a secure, elastic platform, Ravi! 🚀** Start with Topic 1 and learn how to bend the scheduler to your will.
