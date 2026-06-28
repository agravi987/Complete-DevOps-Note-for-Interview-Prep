# 💾 Advanced Storage (CSI & Snapshots)

> **Hey Ravi! 👋** We learned about PVs and PVCs in Phase 2. But under the hood, how does Kubernetes actually know how to talk to AWS, Google Cloud, and VMware to create those hard drives? Does the K8s source code have all those cloud providers hardcoded into it? It used to... until **CSI** came along! Let's get advanced. 💽

---

## 🧠 What is CSI (Container Storage Interface)?

> **CSI is a universal standard that allows storage vendors (AWS, NetApp, Ceph) to write plugins for Kubernetes without modifying the core Kubernetes code.**

- **Old Way (In-Tree):** The code to talk to AWS EBS was baked directly into Kubernetes. If AWS updated their API, you had to wait for a new Kubernetes version! 🐢
- **New Way (Out-of-Tree / CSI):** Kubernetes just says "Hey CSI Driver, make me a disk." The AWS EBS CSI Driver (installed separately) handles the cloud-specific API calls. 🚀

---

## 🏭 StorageClasses (Dynamic Provisioning)

A `StorageClass` tells K8s *which* CSI driver to use when a user asks for a PVC.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com      # Use the AWS EBS CSI Driver!
parameters:
  type: gp3                       # Give me the fast SSDs
  encrypted: "true"               # Encrypt data at rest
```
Now, when a developer creates a PVC and sets `storageClassName: fast-ssd`, the CSI driver instantly provisions a `gp3` encrypted disk in AWS!

---

## 📸 Volume Snapshots (Backups!)

Before CSI, taking a backup of a K8s volume was a nightmare. Now, CSI drivers support **VolumeSnapshots**!

Just like PV and PVC, Snapshots use a two-part system:
1. **VolumeSnapshotClass:** The factory config (Admin creates).
2. **VolumeSnapshot:** The request to take a backup (Dev creates).

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: mysql-backup-monday
spec:
  volumeSnapshotClassName: csi-aws-vsc
  source:
    persistentVolumeClaimName: mysql-pvc  # The PVC we want to backup!
```
*Applying this YAML literally triggers an AWS EBS Snapshot in the cloud!* 🌩️

---

## 🔀 Block vs File vs Object Storage

Interviewers love this! Know the difference:

1. **Block Storage (AWS EBS, GCP Persistent Disk)**
   - **Access Mode:** ReadWriteOnce (RWO). 
   - **Use Case:** Databases (MySQL, Postgres). Extremely fast, but can only attach to ONE node at a time.
   
2. **File Storage (AWS EFS, NFS)**
   - **Access Mode:** ReadWriteMany (RWX).
   - **Use Case:** Shared web assets, CMS uploads (WordPress). Multiple pods on multiple nodes can read/write simultaneously. Slower than block.

3. **Object Storage (AWS S3)**
   - **Access Mode:** Not usually mounted via PVs! (Though CSI drivers for S3 exist now, it's rare).
   - **Use Case:** Apps use the AWS SDK in their code to upload/download files.

---

## 👻 StatefulSets + PVC Templates

Remember StatefulSets from Phase 2? This is why they are magic.
Instead of referencing a single PVC, a StatefulSet uses a `volumeClaimTemplate`.

If `replicas: 3`, the StatefulSet controller talks to the StorageClass/CSI driver **3 separate times**, provisioning 3 distinct cloud disks, and maps one to each pod (`pod-0`, `pod-1`, `pod-2`). If `pod-1` moves to a new node, the CSI driver unmounts disk 1 and remounts it on the new node automatically!

---

## 🎤 Interview Knockout Answers

**Q: What is the Container Storage Interface (CSI) and why was it introduced?**
> "CSI is an open standard API that allows storage vendors to develop plugins for Kubernetes independently of the core Kubernetes codebase (moving from 'in-tree' to 'out-of-tree' plugins). This allows vendors to release updates, bug fixes, and new features faster without waiting for a Kubernetes release cycle."

**Q: Explain the difference between ReadWriteOnce (RWO) and ReadWriteMany (RWX) and give an example of cloud storage for each.**
> "RWO means a volume can be mounted as read/write by only a single Node at a time. This is typical for Block Storage like AWS EBS, which is used for databases. RWX means the volume can be mounted read/write by multiple Nodes simultaneously. This requires File/Network storage like AWS EFS or NFS, commonly used for shared directories."

**Q: How do you backup a Persistent Volume in Kubernetes natively?**
> "If your underlying CSI driver supports it, you can use the VolumeSnapshot API. You create a `VolumeSnapshot` object that references a target PVC. The CSI driver intercepts this request and triggers a native snapshot in the cloud provider (like an EBS Snapshot in AWS)."

---

## ⚡ 30-Second Revision

- 🔌 **CSI** = Out-of-tree plugins connecting K8s to Cloud storage APIs.
- 🏭 **StorageClass** = Tells K8s which CSI driver to use to dynamically create disks.
- 📸 **VolumeSnapshots** = Native K8s API to trigger cloud disk backups.
- 🧱 **Block (EBS)** = Fast, 1 node only (RWO). Used for DBs.
- 📁 **File (EFS/NFS)** = Shared, many nodes (RWX). Used for shared folders.
- 👻 **StatefulSets** use `volumeClaimTemplates` to create unique disks per replica.

> Next → [CNI & Networking](07-cni-networking.md) 🚀
