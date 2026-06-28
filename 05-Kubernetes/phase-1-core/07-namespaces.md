# 🏠 Namespaces — Rooms in the Kubernetes House

> **Ravi! 👋** Imagine your company has 10 teams all using the same Kubernetes cluster. Without Namespaces, it would be chaos — resources mixing up, teams stepping on each other. Namespaces are the solution: **organized rooms in one big house.** 🏡

---

## 🧠 What is a Namespace?

> **A Namespace is a logical partition inside a Kubernetes cluster. It separates resources by team, environment, or app — without needing separate clusters.**

**Think of it as:**
```
One Kubernetes Cluster
├── namespace: dev      ← Dev team works here
├── namespace: staging  ← QA validates here
├── namespace: prod     ← Real users hit this
└── namespace: tools    ← Monitoring, logging tools
```

Resources in the same namespace can see each other easily.
Resources in different namespaces need extra configuration to communicate.

---

## 🏢 The Office Building Analogy

```
🏢 Kubernetes Cluster = Office Building
   │
   ├── 🚪 Floor 1 (dev namespace)
   │     └── dev-team pods, services, configs
   │
   ├── 🚪 Floor 2 (staging namespace)
   │     └── QA pods, services, configs
   │
   └── 🚪 Floor 3 (prod namespace)
         └── Production pods, services, configs

Everyone shares the building (same cluster infra)
but each floor has its own space. Security can be floor-specific.
```

---

## 🏷️ The 4 Default Namespaces (Know These!)

| Namespace | Purpose |
|---|---|
| `default` | Where your stuff goes if you don't specify a namespace |
| `kube-system` | Kubernetes' own components (don't touch!) |
| `kube-public` | Publicly readable info (cluster info) |
| `kube-node-lease` | Node heartbeat records |

> 💡 **Ravi's Tip:** Never put your apps in `kube-system`. That's Kubernetes' sacred space. 🚫

---

## 📄 Namespace YAML

```yaml
# Create a namespace
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    environment: development
    team: backend
```

---

## 📄 Creating Resources IN a Namespace

**Method 1: In the YAML file:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  namespace: dev   # ← Specify it here!
spec:
  containers:
  - name: app
    image: nginx:1.27
```

**Method 2: With kubectl flag:**
```bash
kubectl apply -f pod.yaml -n dev
```

---

## 🎮 Namespace Commands

```bash
# List all namespaces
kubectl get namespaces     # or kubectl get ns

# Create namespace quickly
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod

# Get pods in a specific namespace
kubectl get pods -n dev

# Get ALL pods across ALL namespaces
kubectl get pods -A        # -A = --all-namespaces

# Switch your default namespace (so you don't type -n every time)
kubectl config set-context --current --namespace=dev

# Check which namespace you're in
kubectl config view --minify | grep namespace

# Delete a namespace (⚠️ deletes EVERYTHING in it!)
kubectl delete namespace dev
```

---

## 🌐 Cross-Namespace Communication

Namespaces are NOT full network isolation. Services can still talk across namespaces using the full DNS name:

```
Within same namespace:
  http://web-service

Across namespaces:
  http://web-service.dev.svc.cluster.local
           │         │   │    │
           │         │   │    └── (always)
           │         │   └── svc (service)
           │         └── dev (namespace)
           └── web-service (service name)
```

> 💡 **Ravi, memorize this pattern:** `<service>.<namespace>.svc.cluster.local`
> Interviewers LOVE asking about cross-namespace communication!

---

## ⚡ Resource Quotas per Namespace

Namespaces can have limits (ResourceQuota). This prevents one team from eating all cluster resources:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "10"           # Max 10 pods
    requests.cpu: "4"    # Max 4 CPU cores requested
    requests.memory: 8Gi # Max 8GB RAM requested
```

---

## 🆚 Namespaces vs Labels

| | Namespace | Label |
|---|---|---|
| **Purpose** | Organize and isolate resources | Tag and filter resources |
| **Scope** | Cluster-level grouping | Object-level metadata |
| **Network effect** | Affects DNS names | No network effect |
| **Quota** | Can set resource quotas | Can't set quotas |
| **When to use** | Separate environments/teams | Mark app/version/tier |

---

## 🎤 Interview Knockout Answers

**Q: What is a namespace?**
> "A namespace is a logical partition inside a Kubernetes cluster. It provides scope for names — resources in different namespaces can have the same name without conflict. Namespaces are used to separate teams, environments, or applications on the same cluster."

**Q: Is a namespace a security boundary?**
> "Not by default! Namespaces provide name isolation but NOT network isolation. Pods in different namespaces can communicate unless you add NetworkPolicies. For security boundaries, combine namespaces with RBAC and NetworkPolicies."

**Q: How do services communicate across namespaces?**
> "Using the full DNS name format: `<service-name>.<namespace>.svc.cluster.local`. Within the same namespace, just the service name is enough."

---

## ⚡ 30-Second Revision

```
🏠 Namespace = logical partition in a cluster
🔹 default, kube-system, kube-public, kube-node-lease
📋 -n <namespace> flag in every kubectl command
🌐 Cross-namespace DNS: svc.namespace.svc.cluster.local
⚠️  NOT a security boundary alone (add NetworkPolicy!)
📊 Add ResourceQuota to limit resources per namespace
🎯 Use: dev / staging / prod / team-name
```

---

> **Ravi, well done! 🎉** Namespaces will appear everywhere in real clusters. It's essential. Next → **Labels and Selectors** — the glue that connects everything! [08-labels-selectors.md](08-labels-selectors.md) 🚀
