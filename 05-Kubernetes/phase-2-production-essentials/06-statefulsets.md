# 🗄️ StatefulSets — For Apps That Matter

> **Hey Ravi! 👋** Deployments treat all Pods exactly the same — like identical clones. If one dies, another pops up. Who cares? But what if your Pod is a **Database Master node**, and the other is a **Slave node**? They aren't clones! They have unique identities and unique data. For these special snowflakes, we use **StatefulSets**. ❄️

---

## 🧠 What is a StatefulSet?

> **A StatefulSet manages stateful applications. It provides guarantees about the ordering and uniqueness of Pods. Each Pod gets a permanent name, a permanent network ID, and its very own persistent storage.**

---

## 🐄 Cattle vs Pets Analogy (Famous DevOps Concept!)

- **Deployments are Cattle:** 🐄 If a cow gets sick, you don't take it to the vet, you just get a new cow. Pod names are random (`web-84j9x`). If one dies, it's replaced by a new random string (`web-99y2z`). They share data (or have no data).
- **StatefulSets are Pets:** 🐕 Your pet has a name (Fido). If Fido gets sick, you nurse Fido back to health! Pod names are strict and ordered (`db-0`, `db-1`, `db-2`). If `db-1` dies, the replacement is EXACTLY `db-1`, and it reattaches to `db-1`'s specific hard drive!

---

## 🏆 The 3 Superpowers of StatefulSets

### 1️⃣ Stable Network Identity
Instead of random hashes, pods get sticky, ordered names:
- `mysql-0`
- `mysql-1`
- `mysql-2`
They also get predictable DNS names: `mysql-0.mysql-svc.default.svc.cluster.local`

### 2️⃣ Ordered Deployment & Scaling
They start strictly in order: `0` first. Wait until it's running. Then `1`. Then `2`.
They terminate in reverse order: `2`, then `1`, then `0`. 
*(Great for databases where `0` must be the Master!)*

### 3️⃣ Stable Storage (volumeClaimTemplates)
If `mysql-1` dies, K8s spins up a new `mysql-1` and safely attaches the EXACT SAME Persistent Volume that the old `mysql-1` was using. No data loss!

---

## 👻 The Headless Service (Required!)

StatefulSets require a special Service called a **Headless Service**.
Normally, a Service load-balances across all pods. But in a database, I don't want a random pod — I specifically need to talk to `mysql-0` (the Master) to write data!

A Headless Service sets `clusterIP: None`. It doesn't load balance. It just creates DNS records for every single pod individually.

---

## 📄 Real YAML (Headless Svc + StatefulSet)

```yaml
# 1. The Headless Service
apiVersion: v1
kind: Service
metadata:
  name: mysql-hsvc
spec:
  clusterIP: None       # THIS MAKES IT HEADLESS! 👻
  selector:
    app: mysql
  ports:
  - port: 3306

---
# 2. The StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql-hsvc" # Must link to the headless service!
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql

  # THIS IS THE MAGIC! Every pod gets its own PVC!
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

---

## 🎮 Commands

```bash
# Get statefulsets
kubectl get sts

# Watch them start up one by one!
kubectl get pods -w
# Output:
# mysql-0   Pending
# mysql-0   Running
# mysql-1   Pending   <-- Doesn't start until mysql-0 is Ready!

# Test DNS resolution (from another pod)
nslookup mysql-0.mysql-hsvc
```

---

## 🎤 Interview Knockout Answers

**Q: When would you use a StatefulSet instead of a Deployment?**
> "You use a StatefulSet for applications that require persistent, unique identities and stable storage per replica. Common use cases are databases (MySQL, Cassandra), message queues (Kafka), or consensus systems (ZooKeeper). If the application is stateless and replicas are interchangeable, you use a Deployment."

**Q: What is a Headless Service and why do StatefulSets need them?**
> "A Headless Service has `clusterIP: None`. Instead of load-balancing traffic, it creates individual DNS records for every Pod in the StatefulSet. This is required because stateful applications often need to communicate with specific peers (e.g., a web app explicitly sending write queries to the 'Master' pod `db-0`)."

**Q: Explain how volumeClaimTemplates work.**
> "Instead of all Pods sharing a single PVC, `volumeClaimTemplates` dynamically creates a brand new, unique PVC (and PV) for every single Pod in the StatefulSet. If a Pod dies, it is rescheduled and reattached to its exact same unique volume."

---

## ⚡ 30-Second Revision

- 🗄️ **StatefulSet** = For databases, Kafka, stateful apps.
- 🐕 **Pets, not cattle.** Pods get ordered names (`pod-0`, `pod-1`).
- 👻 **Headless Service** (`clusterIP: None`) = gives DNS to individual pods.
- 💾 **volumeClaimTemplates** = gives every pod its own personal hard drive.
- 🔢 **Strict Ordering:** Starts 0, 1, 2. Deletes 2, 1, 0.

> Next → [DaemonSets](07-daemonsets.md) 🚀
