# 🌱 Phase 1 — Core Kubernetes

> **Hey Ravi! 👋 Welcome to the foundation.** This is where Kubernetes goes from scary buzzword to something you can actually explain at a whiteboard. Take your time here. Every concept in Phase 2-5 builds on this. 🧱

---

## 🧠 What You'll Be Able to Do After Phase 1

By the end of these 13 topics, you will:

- ✅ Explain Kubernetes architecture to any interviewer with confidence
- ✅ Read and write real YAML manifests from scratch
- ✅ Debug broken pods like a pro
- ✅ Understand how traffic flows in a cluster
- ✅ Know the difference between Pods, ReplicaSets, and Deployments (and when to use each)
- ✅ Manage configuration and secrets properly

---

## 📚 The 13 Topics — Go In Order!

| # | Topic | What You'll Learn | Vibe |
|---|---|---|---|
| 1 | [☸️ Kubernetes Architecture](01-kubernetes-architecture.md) | The big picture: control plane + nodes | 🗺️ The map |
| 2 | [🧩 Components](02-components.md) | Every component and its exact job | 🎭 Meet the cast |
| 3 | [🫧 Pods](03-pods.md) | The atomic unit — everything runs in Pods | ⚛️ The atom |
| 4 | [🤝 Multi-Container Pods](04-multi-container-pods.md) | Sidecar, Ambassador, Adapter patterns | 🧑‍🤝‍🧑 Buddy system |
| 5 | [🔁 ReplicaSets](05-replicasets.md) | Auto-healing — keeps pods alive | 💂 The bodyguard |
| 6 | [🚀 Deployments](06-deployments.md) | Rolling updates + rollback = production king | 👑 The king |
| 7 | [🏠 Namespaces](07-namespaces.md) | Organize your cluster into rooms | 🏢 The office |
| 8 | [🏷️ Labels & Selectors](08-labels-selectors.md) | The glue that connects everything | ⚗️ Chemical bonds |
| 9 | [🌐 Services](09-services.md) | Stable network access to your Pods | 🛎️ Front desk |
| 10 | [⌨️ kubectl Commands](10-kubectl-commands.md) | Your daily toolkit — bookmark this! | 🎮 Remote control |
| 11 | [📄 YAML Manifests](11-yaml-manifest-files.md) | The language of Kubernetes | 🗣️ How to talk to K8s |
| 12 | [🧾 ConfigMaps](12-configmaps.md) | Externalize non-sensitive config | 📝 Sticky notes |
| 13 | [🔐 Secrets](13-secrets.md) | Secure sensitive data (passwords, tokens) | 🗝️ The locked safe |

---

## 💡 Study Tips from Your Best Friend

> **1. Problem → Solution → YAML → Commands**
> Always understand WHY the thing exists before HOW to write its YAML.

> **2. Draw diagrams**
> Sketch the relationship between Deployment → ReplicaSet → Pod → Service on paper. Once it clicks visually, it stays forever.

> **3. Break things intentionally**
> Delete a pod → watch the ReplicaSet recreate it. Update a Deployment → watch rolling update happen. Nothing teaches like watching it live.

> **4. One topic per session**
> Don't try to finish all 13 in one day. One per session with hands-on practice works better. 🧠

> **5. Interview prep as you go**
> Each file has "Interview Knockout Answers." Read these out loud — saying it out loud is 10x more effective than just reading.

---

## 🎨 Visual References

- [☸️ Kubernetes Architecture Diagram](../../assets/topic-summaries/kubernetes-architecture.svg)
- [🫧 Pods and ReplicaSets](../../assets/topic-summaries/pods-replicasets.svg)
- [🚀 Kubernetes Deployments](../../assets/topic-summaries/kubernetes-deployments.svg)
- [🌐 Services and Networking](../../assets/topic-summaries/kubernetes-services-networking.svg)
- [🔄 Rolling Updates](../../assets/topic-summaries/kubernetes-rolling-updates.svg)

---

## 🏁 Progress Tracker

Track your progress here (edit this file!):

- [ ] 01 - Architecture
- [ ] 02 - Components
- [ ] 03 - Pods
- [ ] 04 - Multi-Container Pods
- [ ] 05 - ReplicaSets
- [ ] 06 - Deployments ⭐ (most important!)
- [ ] 07 - Namespaces
- [ ] 08 - Labels & Selectors
- [ ] 09 - Services ⭐ (second most important!)
- [ ] 10 - kubectl Commands
- [ ] 11 - YAML Manifests
- [ ] 12 - ConfigMaps
- [ ] 13 - Secrets

---

> **Let's go Ravi! 🚀** Start with Topic 1 and don't look back. The cluster awaits!
