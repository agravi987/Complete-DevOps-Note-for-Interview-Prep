# 🗄️ etcd Deep Dive

> **Hey Ravi! 👋** If Kubernetes is the brain, `etcd` is the memory. It is the SINGLE source of truth for the entire cluster. If a Pod is running but it's not written in `etcd`, as far as Kubernetes is concerned, it doesn't exist. If `etcd` dies, the cluster dies. Let's learn how to protect the most important component in K8s! 💎

---

## 🧠 What is etcd?

> **etcd is a distributed, consistent, highly available key-value store. It stores the entire configuration, state, and metadata of the Kubernetes cluster.**

🚨 **CRITICAL FACT:** Only ONE component is allowed to talk to `etcd` — the `kube-apiserver`. No other component (scheduler, kubelet, controller) is allowed to touch it directly!

---

## 🗳️ The Raft Consensus Algorithm

How does `etcd` stay Highly Available? You run it on 3 or 5 Master nodes. But how do 3 databases keep their data perfectly synced without overwriting each other?

They use the **Raft Consensus Algorithm**:
1. **Leader Election:** The 3 etcd nodes vote for a Leader.
2. **Writes:** If the API Server wants to write data, it MUST go to the Leader.
3. **Replication:** The Leader sends the data to the Followers.
4. **Quorum:** The Leader waits until a *majority* of nodes acknowledge the write before telling the API server "Success!"

> **Quorum Formula:** `(N / 2) + 1`
> In a 3-node cluster, Quorum is `2`. You can lose 1 node and survive!
> In a 5-node cluster, Quorum is `3`. You can lose 2 nodes and survive!
> *(Always run an ODD number of Master nodes!)*

---

## 🗑️ Compaction and Defragmentation

`etcd` is a versioned database. If you change a Deployment's replica count from 1 to 2, `etcd` doesn't overwrite the row. It saves a *new* revision.

This is great for `kubectl rollout undo` (history!), but eventually, `etcd` will run out of space! (Max size is usually 2GB).

1. **Compaction:** `kube-apiserver` tells `etcd` every 5 minutes: "Delete all historical revisions older than X minutes."
2. **Defragmentation:** Even after compaction, there are "holes" in the database file. Admins must occasionally run `etcdctl defrag` to compress the file and win back physical disk space.

---

## 💾 Backup and Restore (The Lifesaver)

If you lose `etcd` quorum, your cluster is dead. The only way back is a restore.

**Taking a Backup (Snapshot):**
```bash
# You must provide the certs to prove you are an admin!
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/backup/etcd-snapshot.db
```

**Restoring:**
When restoring, you must stop the `kube-apiserver`, clear the old etcd data directory, run `etcdctl snapshot restore`, and then restart the cluster!

---

## 🔒 Encryption at Rest

By default, `etcd` stores your Kubernetes **Secrets** in plain base64 text on the hard drive! If someone steals the hard drive from the AWS data center, they have your database passwords. 😱

You must enable **EncryptionConfiguration** in the API Server. This intercepts Secrets before they are sent to `etcd` and encrypts them using a KMS (Key Management Service) provider (like AWS KMS) or a local key.

---

## 🏎️ Performance Considerations

`etcd` writes every single transaction to a Write-Ahead Log (WAL) on disk. If the disk is slow, `etcd` slows down, and the ENTIRE Kubernetes cluster slows down.

> **Golden Rule:** ALWAYS put `etcd` on **SSD (Solid State Drives)**. Never use spinning HDDs. Also, avoid running heavy logging or monitoring tools on the Master nodes, as they will compete with `etcd` for disk IOPS.

---

## 🎤 Interview Knockout Answers

**Q: Explain how etcd ensures High Availability in a multi-master Kubernetes cluster.**
> "etcd uses the Raft consensus algorithm. In a 3-node setup, one node is elected Leader and handles all writes, replicating them to the Followers. A write is only considered successful when a quorum—a majority of nodes—acknowledges it. For 3 nodes, the quorum is 2, meaning the cluster can survive the loss of 1 node without data inconsistency or downtime."

**Q: Why do we always deploy an ODD number of master nodes (e.g., 3 or 5)?**
> "Because of how quorum works in Raft. A 3-node cluster has a quorum of 2 (can survive 1 failure). A 4-node cluster has a quorum of 3 (can STILL only survive 1 failure). Adding a 4th node gives you no extra fault tolerance, but it slows down writes because the Leader now has to wait for 3 acknowledgments instead of 2. Therefore, you always jump from 3 to 5 nodes."

**Q: Are Kubernetes Secrets encrypted in etcd by default?**
> "No, by default, Secrets are only base64 encoded, not encrypted. If an attacker gains access to the etcd data file on the disk, they can read all the secrets. To secure them, you must configure the `kube-apiserver` with an `EncryptionConfiguration` file to encrypt the data at rest before it is sent to etcd."

---

## ⚡ 30-Second Revision

- 🗄️ **etcd** = Distributed key-value store holding the entire cluster state.
- 🔒 **API Server** = The ONLY component allowed to talk directly to etcd.
- 🗳️ **Raft / Quorum** = Needs `(N/2)+1` nodes alive to function. (Run 3 or 5 nodes!).
- 💾 **Snapshot** = Use `etcdctl snapshot save` to backup the cluster state.
- 🔐 **Encryption** = Secrets are NOT encrypted by default! Enable EncryptionConfiguration.
- ⚡ **SSDs Only** = etcd is highly dependent on fast disk IOPS.

> Next → [Networking Internals](04-networking-internals.md) 🚀
