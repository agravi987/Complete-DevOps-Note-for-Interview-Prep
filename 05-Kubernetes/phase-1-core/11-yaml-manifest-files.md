# 📄 YAML Manifest Files — The Language of Kubernetes

> **Ravi! 👋** If you want to speak to Kubernetes, you speak in **YAML.** Every single thing you create — pods, deployments, services, configs — starts as a YAML file. Master this syntax once and you've got the key to the whole Kubernetes kingdom! 🗝️

---

## 🧠 What is a Manifest?

> **A YAML manifest is a file that describes the DESIRED STATE of a Kubernetes object. You give it to the cluster → the cluster makes it real.**

```
You write:    "I want a Deployment with 3 replicas running nginx"
              ↓ (kubectl apply -f deployment.yaml)
Kubernetes:   "Got it! Creating 3 pods with nginx right now..." ✅
```

---

## 🍳 The Recipe Analogy

```
YAML Manifest = Recipe
Kubernetes    = Chef

You write the recipe (YAML)
You hand it to the chef (kubectl apply)
Chef makes exactly what you described
If you change the recipe → Chef updates the dish
You store the recipe in Git → repeatability forever!
```

---

## 📐 The 4 Fields Every Manifest Has

```yaml
apiVersion:   # Which API group and version
kind:         # What TYPE of object you're creating
metadata:     # Name, namespace, labels, annotations
spec:         # The DESIRED STATE — what you actually want
```

> 💡 **Memorize this structure!** It's the same for EVERY Kubernetes object. Only `spec` changes based on the `kind`.

---

## 🗂️ Which apiVersion for Which Kind?

| Kind | apiVersion |
|---|---|
| Pod | `v1` |
| Service | `v1` |
| ConfigMap | `v1` |
| Secret | `v1` |
| Namespace | `v1` |
| Deployment | `apps/v1` |
| ReplicaSet | `apps/v1` |
| StatefulSet | `apps/v1` |
| DaemonSet | `apps/v1` |
| Job | `batch/v1` |
| CronJob | `batch/v1` |
| Ingress | `networking.k8s.io/v1` |
| HPA | `autoscaling/v2` |

> 💡 **Tip:** If you forget, run `kubectl api-resources | grep Deployment` to find the correct apiVersion!

---

## 📄 Full Real-World Deployment Manifest (Annotated)

```yaml
# ─────────────────────────────────────────────────
# HEADER: What object is this?
# ─────────────────────────────────────────────────
apiVersion: apps/v1     # Deployments live in apps/v1
kind: Deployment

# ─────────────────────────────────────────────────
# METADATA: Identity & tags
# ─────────────────────────────────────────────────
metadata:
  name: web-app               # Object name (must be unique in namespace)
  namespace: prod             # Which namespace
  labels:
    app: web-app              # Labels for filtering/selection
    version: "2.1"
  annotations:
    deployment.kubernetes.io/revision: "3"
    owner: "ravi@company.com"

# ─────────────────────────────────────────────────
# SPEC: What you want to exist
# ─────────────────────────────────────────────────
spec:
  replicas: 3

  selector:
    matchLabels:
      app: web-app            # Connects deployment to its pods

  template:                   # Pod template (spec within spec)
    metadata:
      labels:
        app: web-app          # Must match selector!
    
    spec:
      containers:
      - name: web
        image: nginx:1.27
        
        ports:
        - containerPort: 80

        # Environment variables
        env:
        - name: APP_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_HOST

        # Resource limits (ALWAYS set these in production!)
        resources:
          requests:
            cpu: "100m"       # 0.1 CPU core
            memory: "128Mi"   # 128 megabytes
          limits:
            cpu: "500m"       # 0.5 CPU core
            memory: "256Mi"

        # Health checks
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10

        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
```

---

## 🎮 YAML-Specific Commands

```bash
# Apply (create or update) — idempotent!
kubectl apply -f deployment.yaml

# Apply all files in a directory
kubectl apply -f manifests/

# Preview what WOULD change (dry run — no actual changes!)
kubectl diff -f deployment.yaml

# Validate YAML without applying
kubectl apply -f deployment.yaml --dry-run=client

# See the LIVE YAML of a running resource
kubectl get deployment web-app -o yaml

# Learn what fields an object accepts
kubectl explain deployment.spec
kubectl explain deployment.spec.strategy
kubectl explain pod.spec.containers

# Generate YAML quickly (then edit it)
kubectl create deployment web --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl expose deployment web --port=80 --dry-run=client -o yaml > service.yaml
```

---

## ⚠️ YAML Gotchas — Common Mistakes

### 1. Indentation Errors (Most common!)
```yaml
# ❌ WRONG
spec:
containers:    # Should be indented!
- name: app

# ✅ CORRECT
spec:
  containers:
  - name: app
```

### 2. Tabs vs Spaces
```
❌ YAML does NOT allow TAB characters for indentation
✅ Always use SPACES (2 or 4, be consistent)
```

### 3. String vs Number Confusion
```yaml
# ❌ Port might be read as number, but some fields need strings
version: 1.0         # YAML reads as float 1.0
version: "1.0"       # ✅ Explicitly a string

replicas: "3"        # ❌ Should be number
replicas: 3          # ✅ Number
```

### 4. selector ≠ template.labels
```yaml
# ❌ These don't match!
selector:
  matchLabels:
    app: web
template:
  metadata:
    labels:
      app: web-frontend   # Different! Kubernetes will error.

# ✅ They MUST match
selector:
  matchLabels:
    app: web-app
template:
  metadata:
    labels:
      app: web-app        # Same!
```

---

## 📁 Recommended File Structure

```
my-app/
├── manifests/
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── README.md

Apply order: namespace → configmap → secret → deployment → service → ingress
```

---

## 🎤 Interview Knockout Answers

**Q: What are apiVersion, kind, metadata, and spec?**
> "Every Kubernetes manifest has 4 top-level fields: apiVersion (which API group/version), kind (type of resource), metadata (name, namespace, labels), and spec (the desired state). This structure is universal across all Kubernetes objects."

**Q: Why is indentation important in YAML?**
> "YAML uses indentation to represent hierarchy — it has no brackets like JSON. Incorrect indentation changes the meaning of the document, causing Kubernetes to misinterpret the structure and potentially error or create wrong objects."

**Q: Why do teams prefer YAML for Kubernetes?**
> "YAML is human-readable and can be stored in Git — enabling GitOps workflows where the cluster state is version-controlled. kubectl apply is idempotent, so you can safely apply the same YAML repeatedly. It also serves as documentation of your infrastructure."

---

## ⚡ 30-Second Revision

```
📄 Manifest = YAML file describing desired state
4 SECTIONS: apiVersion / kind / metadata / spec
🔑 kubectl apply -f = create or update (idempotent!)
🧪 --dry-run=client = test without applying
🔍 kubectl explain = learn any field
⚠️  Spaces not tabs! Indentation = structure
📁 Keep manifests in Git for reproducibility
💡 Generate templates: kubectl create --dry-run -o yaml
```

---

> **Ravi, you speak Kubernetes now! 🎉** YAML manifests are everything. Next → **ConfigMaps** — keeping configuration outside your containers! [12-configmaps.md](12-configmaps.md) 🚀
