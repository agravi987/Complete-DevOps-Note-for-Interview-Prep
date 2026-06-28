# 💾 Persistent Volumes (PV & PVC)

> **Hey Ravi! 👋** Remember how Pods are ephemeral and die all the time? Well, if your Pod runs a database and dies, the data inside it vanishes forever! 😱 To stop you from losing your job, Kubernetes separates **Storage** from **Compute** using PVs and PVCs. Let's lock that data down! 🔒

---

## 🧠 What are PV, PVC, and StorageClass?

> **PersistentVolume (PV):** A piece of actual storage in the real world (like an AWS EBS volume or NFS drive).
> **PersistentVolumeClaim (PVC):** A ticket requesting storage (e.g., "I need 10GB of fast storage!").
> **StorageClass (SC):** An automated factory that creates PVs on-the-fly when a PVC asks for one.

---

## 🎟️ The Coat Check Analogy

1. **Storage (The Coat Room):** Where things are actually stored.
2. **PV (The Hanger):** A specific spot to put a coat.
3. **PVC (The Ticket):** You give the front desk a ticket saying "I need a hanger." They give you the ticket, which binds you to that specific hanger.
4. **StorageClass (The Automated Attendant):** If there are no hangers left, the automated attendant builds a new hanger for you instantly!

---

## 🏗️ Static vs Dynamic Provisioning

### Old Way: Static Provisioning 👴
1. K8s Admin creates 10x 100GB PVs manually.
2. Developer creates a PVC for 50GB.
3. K8s matches (binds) the PVC to one of the 100GB PVs. (50GB wasted!)

### Modern Way: Dynamic Provisioning 🚀 (StorageClass)
1. K8s Admin creates a `StorageClass`.
2. Developer creates a PVC for 50GB.
3. The StorageClass talks to AWS/GCP, creates exactly a 50GB disk, creates the PV, and binds it automatically!

---

## 📄 Real YAML (Dynamic Provisioning)

### 1. The Claim (Developer writes this)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce        # Important! See below.
  resources:
    requests:
      storage: 10Gi        # I want 10 GB
  storageClassName: standard # Use the 'standard' StorageClass factory
```

### 2. The Pod (Using the Claim)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - name: mysql
    image: mysql:8.0
    volumeMounts:
    - mountPath: /var/lib/mysql   # Where inside container?
      name: data-volume
  
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: mysql-pvc        # Attach the PVC ticket!
```

---

## 🚦 Access Modes (Super Important!)

When you ask for storage, you must specify HOW it will be accessed:

| Access Mode | Code | Meaning | Use Case |
|---|---|---|---|
| **ReadWriteOnce** | `RWO` | Mounted by ONE node at a time (Read/Write) | Databases, AWS EBS block storage |
| **ReadWriteMany** | `RWX` | Mounted by MANY nodes at once (Read/Write) | Shared files, NFS, AWS EFS |
| **ReadOnlyMany** | `ROX` | Mounted by MANY nodes (Read Only) | Serving static config/assets |

> 🚨 **Gotcha:** You can't use AWS EBS (block storage) with `ReadWriteMany`. Block storage can only attach to ONE machine at a time!

---

## ♻️ Reclaim Policies (What happens when PVC is deleted?)

If I delete my `mysql-pvc`, what happens to the underlying disk (PV) and my data?

1. **Retain:** The PV is kept, but released. Admin must manually clean it up. (Safest for DBs!)
2. **Delete:** The PV and the actual AWS/GCP disk are permanently deleted! 🔥 (Default for dynamically provisioned storage!).
3. **Recycle:** Scrub the data (`rm -rf /thevolume/*`) and make it available again. (Deprecated).

---

## 📦 Other Volume Types (No PV/PVC needed)

Sometimes you don't need persistent cloud storage!

### 1. `emptyDir` (Ephemeral)
- Starts empty when pod starts.
- Erased forever when pod dies.
- Good for: Temporary scratch space, cache, sharing files between two containers in the same Pod.

### 2. `hostPath` (Node-level)
- Mounts a folder directly from the Worker Node's filesystem (e.g., `/var/log`).
- Good for: DaemonSets (like fluentd) that need to read node logs.
- Bad for: Normal apps! (If pod moves to another node, data is left behind).

---

## 🎮 Commands

```bash
# Check your PVCs
kubectl get pvc

# Check the actual Volumes
kubectl get pv

# See available StorageClasses
kubectl get sc

# Is my PVC stuck in "Pending"? Let's find out why!
kubectl describe pvc mysql-pvc
# Usually means: StorageClass missing, or requested size not available.
```

---

## 🎤 Interview Knockout Answers

**Q: Explain the relationship between PV, PVC, and StorageClass.**
> "A PV is the actual physical storage volume. A PVC is a request by a user for storage. A StorageClass enables dynamic provisioning; when a user creates a PVC, the StorageClass automatically talks to the cloud provider, provisions the physical disk, creates the PV, and binds it to the PVC."

**Q: What is the difference between ReadWriteOnce and ReadWriteMany?**
> "ReadWriteOnce (RWO) means the volume can only be mounted as read/write by a single Node at a time, which is typical for block storage like AWS EBS. ReadWriteMany (RWX) means it can be mounted read/write by multiple Nodes simultaneously, which requires file storage like NFS or AWS EFS."

**Q: If I delete a Pod, what happens to its PVC and PV?**
> "If you delete the Pod, the PVC and PV remain completely intact and the data is safe. You can spin up a new Pod and attach it to the same PVC. The fate of the PV only comes into play if you delete the *PVC*, which depends on the Reclaim Policy (Retain or Delete)."

---

## ⚡ 30-Second Revision

- 💾 **PV** = The actual disk.
- 🎟️ **PVC** = Your request to use a disk.
- 🏭 **StorageClass** = The robot that auto-creates disks (Dynamic Provisioning).
- 🔓 **RWO** = One Node. **RWX** = Many Nodes.
- ♻️ **Reclaim Policy:** Delete (disk goes bye-bye) vs Retain (disk stays safely).
- 🗑️ **emptyDir** = Temp storage, dies with the Pod.
- 🖥️ **hostPath** = Maps directly to the Node's filesystem.

> Next → [Jobs & CronJobs](05-jobs-cronjobs.md) 🚀
