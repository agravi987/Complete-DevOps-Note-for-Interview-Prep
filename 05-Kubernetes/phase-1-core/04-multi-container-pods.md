# 🤝 Multi-Container Pods — The Buddy System

> **Ravi! 👋** Most of the time, one container per Pod is enough. But sometimes, your app needs a helper. That's multi-container Pods — two containers sharing a home, working as a team. Let's see why and when! 🏡

---

## 🧠 Why Would You Put 2+ Containers in One Pod?

**When containers need to:**
- 🔗 Share files (same volume)
- 💬 Talk over localhost (super fast, no network hop)
- 👶 Start together and die together (same lifecycle)

> **Golden Rule:** If two containers are so tightly coupled that they CANNOT run independently → put them in the same Pod.

---

## 🧑‍🤝‍🧑 The Roommate Analogy

```
🏠 Pod (Shared Apartment)
   ├── 🧑 Container A — Main App (does the heavy lifting)
   └── 🧑 Container B — Sidecar (helps, supports, assists)
        They share:
        ├── 📡 Same IP address
        ├── 💬 localhost communication
        └── 📁 Shared Volume (like a shared kitchen)
```

---

## 🎭 The 3 Multi-Container Patterns — MUST KNOW!

### 1️⃣ Sidecar Pattern 🛵
> "I enhance the main container"

**Use case:** Log collector, monitoring agent, config watcher

```
Main App → writes logs to /var/log/app.log
Sidecar  → reads /var/log/app.log → sends to Elasticsearch
```

```yaml
containers:
- name: app
  image: my-web-app:v2
  volumeMounts:
  - name: logs
    mountPath: /var/log
- name: log-collector
  image: fluentd:latest
  volumeMounts:
  - name: logs
    mountPath: /var/log  # Same volume! Reads what app writes
```

---

### 2️⃣ Ambassador Pattern 🤝
> "I proxy requests on behalf of the main container"

**Use case:** Local proxy, API gateway, TLS terminator

```
Main App → talks to "localhost:9090" (simple!)
Ambassador → forwards to the actual database (handles complexity)
```

This is cool because the main app doesn't need to know about the database's actual address! 🎯

---

### 3️⃣ Adapter Pattern 🔌
> "I transform the main container's output"

**Use case:** Format logs from app format → to standard format, metric translation

```
Main App → outputs metrics in custom format
Adapter  → translates to Prometheus format
```

---

## 📄 Full Multi-Container YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-demo
  labels:
    app: demo
spec:
  containers:
  
  # Main application container
  - name: app
    image: nginx:1.27
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx

  # Sidecar: reads nginx logs and tails them
  - name: log-sidecar
    image: busybox
    command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx  # Same volume — shared!

  volumes:
  - name: shared-logs
    emptyDir: {}   # Temporary shared storage
```

---

## 🎮 Commands for Multi-Container Pods

```bash
# See logs of SPECIFIC container (-c flag is key!)
kubectl logs multi-container-demo -c app
kubectl logs multi-container-demo -c log-sidecar

# Exec into a SPECIFIC container
kubectl exec -it multi-container-demo -c app -- /bin/bash
kubectl exec -it multi-container-demo -c log-sidecar -- sh

# Describe Pod — shows all container statuses
kubectl describe pod multi-container-demo
```

> 💡 **Ravi's Tip:** When you have multiple containers, **always use -c <container-name>** in logs and exec. Otherwise kubectl will pick the first one (or complain)!

---

## 🆚 Multi-Container Pod vs Separate Pods

| Situation | Multi-Container Pod | Separate Pods |
|---|---|---|
| Containers share files? | ✅ Use same Pod | ❌ Need shared PVC |
| Need localhost comm? | ✅ Use same Pod | ❌ Need Service |
| Scale independently? | ❌ Can't — they scale together | ✅ Yes! |
| Independent lifecycle? | ❌ Start/stop together | ✅ Independent |
| Best example | App + log shipper | Frontend + Backend |

---

## 🎤 Interview Knockout Answers

**Q: Why would you use more than one container in a Pod?**
> "When containers are tightly coupled — they share the same lifecycle, need to share files via a volume, or communicate via localhost at high speed. Common examples: sidecar log collectors, proxy ambassadors, or metric adapters."

**Q: What is a sidecar container?**
> "A sidecar is a helper container in the same Pod as the main app. It enhances or supports the main container — common use cases include log shipping, proxy handling, config watching, and monitoring agents."

**Q: Do containers in one Pod share an IP?**
> "Yes! All containers in a Pod share the same network namespace — same IP, same port space. They communicate via localhost. This is why you can't have two containers in the same Pod listening on the same port."

---

## ⚡ 30-Second Revision

```
🤝 Multi-container pods = containers sharing lifecycle
📡 Same IP, same network, same volumes
🛵 Sidecar = enhances the main app (logs, metrics)
🤝 Ambassador = proxies for the main app
🔌 Adapter = transforms output format
⚠️  Use -c flag in kubectl logs/exec for specific container
🚫 They scale TOGETHER — don't co-locate unrelated services!
```

---

> **Awesome work Ravi! 🎉** You understand both solo and multi-container Pods now. Next — what keeps multiple Pods alive? Enter the **ReplicaSet!** [05-replicasets.md](05-replicasets.md) 🚀
