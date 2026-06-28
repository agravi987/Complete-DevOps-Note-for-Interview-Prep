# 🚑 High Availability (HA) & Disaster Recovery (DR)

> **Hey Ravi! 👋** If your single Master node catches fire, your entire Kubernetes cluster becomes a zombie. The worker nodes will keep running existing apps, but you can't deploy anything new, you can't auto-scale, and you can't fix anything! 🧟‍♂️ To sleep well at night, we need **High Availability (HA)** and **Disaster Recovery (DR)**. Let's make this cluster bulletproof! 🛡️

---

## 🧠 High Availability (Preventing the Disaster)

HA is about building the cluster so a failure goes unnoticed.

### 1. Control Plane HA (The Brain)
- Run **3 or 5 Master Nodes**. (Need an odd number for `etcd` Raft quorum!).
- Put the `kube-apiserver` behind an external Cloud Load Balancer (like AWS NLB). Worker nodes talk to the LB, not a specific master.
- Spread the Master nodes across different **Availability Zones (AZs)**!

### 2. Worker Node HA (The Muscle)
- Use Cloud Auto Scaling Groups. If a node dies, the cloud spins up a new one.
- Use **PodAntiAffinity**: Ensure your 3 app replicas land on 3 *different* physical nodes in 3 *different* AZs!
- Use **Pod Disruption Budgets (PDB)** to prevent admins from draining too many nodes at once.

---

## 🚑 Disaster Recovery (Fixing the Disaster)

DR is what happens when the asteroid actually hits the data center, and the entire cluster is gone. ☄️

### The 2 Golden Metrics of DR:
1. **RPO (Recovery Point Objective):** How much data are you willing to lose? (e.g., "We can lose the last 15 minutes of DB data").
2. **RTO (Recovery Time Objective):** How long can you be down? (e.g., "We must be back online in 1 hour").

---

## 🛟 Velero (The Kubernetes Backup Lifesaver)

How do you backup an entire cluster? You can't just copy/paste files. You use **Velero** (an open-source tool by VMware).

**What Velero does:**
1. Backs up all your YAML objects (Deployments, Services, RBAC) and stores them as a ZIP file in an AWS S3 bucket.
2. Talks to the CSI Driver to take native Cloud Snapshots of your Persistent Volumes (EBS disks).

**The Recovery Plan (If AWS us-east-1 burns down):**
1. Spin up a brand new, empty cluster in `us-west-2`.
2. Install Velero.
3. Point Velero at the S3 bucket.
4. Run `velero restore create`.
5. Velero recreates all your YAMLs and restores your Cloud Disks. You are back online! 🎉

---

## 🐙 The GitOps DR Shortcut

If you use GitOps (ArgoCD), Disaster Recovery for your *stateless* apps is trivial!

You don't even need to backup YAMLs with Velero! If the cluster burns down, you just spin up a new cluster, install ArgoCD, and point it at your GitHub repo. It will immediately redeploy all your Deployments and Services automatically! 

*(Note: You STILL need Velero to backup your stateful Persistent Volume data!)*

---

## 🌍 Cluster Federation (Karmada / Kubefed)

What if you want to run one app across 3 different clusters in 3 different continents simultaneously?

**Cluster Federation** lets you have one "Host" cluster that manages "Member" clusters. You apply a Deployment to the Host, and it automatically splits the replicas: 5 go to AWS America, 5 go to Azure Europe. If the America cluster dies, it auto-migrates all 10 to Europe! 🤯 *(Warning: This is insanely complex and only used by massive enterprises!)*

---

## 🎤 Interview Knockout Answers

**Q: How do you configure a Highly Available (HA) Control Plane in Kubernetes?**
> "To achieve HA, you must deploy multiple control plane nodes—typically 3 or 5 to maintain `etcd` quorum—and distribute them across different Availability Zones. You then place the `kube-apiserver` behind an external Load Balancer so that worker nodes and administrators have a single, highly available endpoint to communicate with, regardless of individual master node failures."

**Q: Explain how you would achieve Disaster Recovery for a Kubernetes cluster.**
> "For stateless configuration, adopting a GitOps workflow (like ArgoCD) ensures that cluster state can be instantly recreated in a new cluster straight from Git. For stateful data and cluster-level resources, I would use Velero. Velero periodically backs up all Kubernetes objects to an object store (like S3) and coordinates with the CSI driver to take snapshots of Persistent Volumes. In a disaster, Velero can restore both the YAML state and the underlying disk data into a fresh cluster."

**Q: What is the difference between RTO and RPO?**
> "RPO (Recovery Point Objective) defines the maximum acceptable amount of data loss measured in time (e.g., losing the last 1 hour of transactions). RTO (Recovery Time Objective) defines the maximum acceptable amount of downtime before the system is restored (e.g., system must be back online within 4 hours)."

---

## ⚡ 30-Second Revision

- 🧠 **Control Plane HA:** 3/5 Masters, spread across AZs, behind a Load Balancer.
- 💪 **Worker Node HA:** PodAntiAffinity (spread pods out) + PDBs.
- 🛟 **Velero** = The industry standard backup tool for K8s YAMLs + PV Snapshots.
- ⏱️ **RPO** = Data loss tolerance. **RTO** = Downtime tolerance.
- 🐙 **GitOps** makes stateless DR trivial (just point ArgoCD at the new cluster).
- 🌍 **Cluster Federation** = Managing multiple clusters as one giant cluster.

> Next → [Security Hardening](07-security-hardening.md) 🚀
