# 🧠 Phase 4 - Advanced Kubernetes

> **Hey Ravi! 👋** Welcome to the Platform Engineering phase! Phase 3 was about using Kubernetes tools. Phase 4 is about **extending and customizing** Kubernetes itself. We're going under the hood to see how the API server works, how custom controllers automate human tasks, and how networking and storage plug into the core system. 🔧

---

## 🎯 What You'll Be Able to Do After Phase 4

By the end of these 7 topics, you will:
- ✅ Patch YAML natively using Kustomize (no Helm templates needed!)
- ✅ Extend the Kubernetes API by teaching it new words (CRDs)
- ✅ Run complex databases on autopilot using Operators
- ✅ Block insecure deployments at the API door (Admission Controllers)
- ✅ Lock down containers so hackers can't get root access
- ✅ Understand how cloud hard drives are dynamically provisioned (CSI)
- ✅ Understand how Pod IPs and network routing actually work (CNI)

---

## 📚 The 7 Advanced Topics

| # | Topic | What You'll Learn | Vibe |
|---|---|---|---|
| 1 | [🛠️ Kustomize](01-kustomize.md) | Base + Overlay YAML patching | 🍔 Building Burgers |
| 2 | [🏗️ CRDs](02-custom-resource-definitions.md) | Custom Resource Definitions | 📖 The Dictionary |
| 3 | [🤖 Operators](03-operators.md) | CRD + Custom Controller automation | 👨‍🔧 Robot DBA |
| 4 | [🛂 Admission Controllers](04-admission-controllers.md) | Intercepting/Blocking API requests | 🛑 The Bouncer |
| 5 | [🛡️ Pod Security](05-pod-security.md) | SecurityContext and restricted pods | 🔒 Locking the Cage |
| 6 | [💾 Advanced Storage](06-storage-advanced.md) | CSI plugins and Volume Snapshots | 💽 Plug & Play Disks |
| 7 | [🕸️ CNI & Networking](07-cni-networking.md) | Pod IPs, Overlay networks, Calico/Cilium | 📬 The Post Office |

---

## 💡 Study Tips for the Advanced Mindset

> **1. The Operator Pattern is Everything**
> The most important concept in modern Kubernetes is the Operator Pattern. If you understand how a CRD stores data, and how a custom controller runs a reconciliation loop to act on that data, you understand how 90% of modern K8s tools (ArgoCD, Crossplane, Cert-Manager) work.

> **2. In-Tree vs Out-of-Tree (Plugins)**
> Kubernetes has removed storage (CSI) and networking (CNI) from its core source code. Understand *why* they did this (so vendors can update plugins without waiting for K8s version releases).

> **3. Interview Focus: API Request Lifecycle**
> Interviewers love asking what happens when you type `kubectl apply`. Make sure you can trace it: Auth → RBAC → Mutating Webhook → Schema Validation (CRD) → Validating Webhook → Etcd!

---

## 🏁 Progress Tracker

Track your progress here (edit this file!):

- [ ] 01 - Kustomize
- [ ] 02 - Custom Resource Definitions (CRDs) ⭐
- [ ] 03 - Operators ⭐ (Huge interview topic)
- [ ] 04 - Admission Controllers (Webhooks)
- [ ] 05 - Pod Security & SecurityContext
- [ ] 06 - Advanced Storage (CSI)
- [ ] 07 - CNI & Networking

---

> **Time to level up, Ravi! 🚀** Start with Kustomize and let's get building!
