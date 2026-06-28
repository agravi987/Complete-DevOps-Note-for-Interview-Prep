# ⌨️ kubectl Commands — Your Kubernetes Remote Control

> **Ravi! 👋** kubectl is your magic wand. 🪄 You can see everything, fix anything, and debug any problem — all through kubectl. This note is a practical cheat sheet you'll use EVERY SINGLE DAY on the job. Bookmark this one! 📌

---

## 🧠 How kubectl Works

```
YOU → kubectl → kubeconfig → API Server → Kubernetes

kubectl reads ~/.kube/config → finds cluster address + auth
→ sends HTTP request to API Server
→ API Server does the thing
→ You get the result
```

> 💡 **The key file:** `~/.kube/config` — your credentials and cluster connection info. Multiple clusters? Multiple contexts in this file!

---

## 🎮 The 6 Command Groups You Must Know

### 1️⃣ GET — See what's there

```bash
# Basic get commands
kubectl get pods                     # Pods in current namespace
kubectl get pods -n kube-system      # Pods in specific namespace
kubectl get pods -A                  # ALL pods in ALL namespaces
kubectl get pods -o wide             # Extra info: node, IP
kubectl get pods --show-labels       # Show pod labels

kubectl get deployments
kubectl get services                 # or: kubectl get svc
kubectl get replicasets              # or: kubectl get rs
kubectl get namespaces               # or: kubectl get ns
kubectl get configmaps               # or: kubectl get cm
kubectl get secrets
kubectl get nodes
kubectl get endpoints

# Get EVERYTHING in a namespace
kubectl get all
kubectl get all -n prod

# Output formats (very useful!)
kubectl get pod my-pod -o yaml       # Full YAML output
kubectl get pod my-pod -o json       # Full JSON output
kubectl get pods -o jsonpath='{.items[*].metadata.name}'  # Custom field!
```

---

### 2️⃣ DESCRIBE — Deep dive into an object

```bash
kubectl describe pod <pod-name>         # Full pod events + status
kubectl describe deployment <dep-name>  # Deployment conditions
kubectl describe node <node-name>       # Node health + capacity
kubectl describe svc <service-name>     # Service details + endpoints
```

> 💡 **When to use describe:** When something is broken! The `Events` section at the bottom of `describe` output tells you EXACTLY what went wrong. This is your #1 debug tool!

---

### 3️⃣ LOGS — Read what your app says

```bash
kubectl logs <pod-name>                  # Current logs
kubectl logs <pod-name> -f               # Follow/live-tail logs (like tail -f)
kubectl logs <pod-name> --previous       # Logs from crashed previous container
kubectl logs <pod-name> -c <container>   # Specific container in multi-pod
kubectl logs <pod-name> --tail=100       # Last 100 lines only
kubectl logs <pod-name> --since=1h       # Logs from last 1 hour
```

---

### 4️⃣ EXEC — Get inside a pod

```bash
kubectl exec -it <pod-name> -- /bin/bash    # Open bash shell
kubectl exec -it <pod-name> -- /bin/sh      # Open sh shell (minimal images)
kubectl exec -it <pod-name> -- env          # See environment variables
kubectl exec -it <pod-name> -- curl http://web-service  # Test connectivity

# Multi-container: specify which container with -c
kubectl exec -it <pod-name> -c <container> -- /bin/bash
```

---

### 5️⃣ APPLY / CREATE / DELETE — Manage resources

```bash
# Declarative (preferred!)
kubectl apply -f file.yaml              # Create or update
kubectl apply -f directory/             # Apply all files in a folder
kubectl apply -f https://url/file.yaml  # Apply from URL

# See what WOULD change before applying
kubectl diff -f file.yaml

# Delete resources
kubectl delete -f file.yaml             # Delete what the file defines
kubectl delete pod <pod-name>           # Delete specific pod
kubectl delete pod <pod-name> --force   # Force delete (use with care!)
kubectl delete deployment <dep-name>    # Deletes deployment + its pods

# Imperative (for quick ops)
kubectl create deployment web --image=nginx:1.27
kubectl expose deployment web --port=80 --type=NodePort
kubectl scale deployment web --replicas=5
```

---

### 6️⃣ ROLLOUT — Manage deployments

```bash
kubectl rollout status deployment/web      # Is rollout complete?
kubectl rollout history deployment/web     # See all versions
kubectl rollout undo deployment/web        # Rollback!
kubectl rollout restart deployment/web     # Restart all pods (force re-pull)
```

---

## 🔍 Power Commands — Impress Your Interviewer

```bash
# Watch resources update in real-time
kubectl get pods -w                        # -w = watch

# Run a temporary debug pod and delete it on exit
kubectl run debug --image=busybox --rm -it -- sh

# Port forward (test service without external access)
kubectl port-forward pod/my-pod 8080:80
kubectl port-forward svc/my-service 8080:80

# Read the YAML fields definition
kubectl explain pod.spec.containers
kubectl explain deployment.spec.strategy

# Get all resources in a namespace
kubectl api-resources --namespaced=true

# Get resource limits & usage
kubectl top pods                           # CPU + Memory usage
kubectl top nodes

# Copy files to/from a pod
kubectl cp my-pod:/app/logs/app.log ./local-copy.log
kubectl cp local-file.txt my-pod:/tmp/

# Label a resource on the fly
kubectl label pod my-pod environment=prod

# Annotate a resource
kubectl annotate pod my-pod owner="ravi@company.com"

# Check your current context (cluster + namespace)
kubectl config current-context
kubectl config get-contexts                # See all configured clusters
kubectl config use-context production      # Switch to production cluster
```

---

## 🚨 The Debug Workflow — When Something's Broken

```bash
# Step 1: See what's happening
kubectl get pods            # Any pods not Running?

# Step 2: What's the status?
kubectl describe pod <bad-pod>  # Read Events section!

# Step 3: What's the app saying?
kubectl logs <bad-pod>          # App error messages

# Step 4: Previous crash?
kubectl logs <bad-pod> --previous   # If it's restarting

# Step 5: Network issue?
kubectl get endpoints <service>      # Any pods behind service?
kubectl exec -it <good-pod> -- curl http://<service>:80

# Step 6: Config issue?
kubectl describe configmap <cm>
kubectl exec -it <pod> -- env | grep MY_VAR
```

---

## 🆚 Imperative vs Declarative

| | Imperative | Declarative |
|---|---|---|
| **Style** | `kubectl create/run/expose` | `kubectl apply -f file.yaml` |
| **Good for** | Quick experiments, one-offs | Production, GitOps, repeatable |
| **Trackable in Git?** | ❌ No | ✅ Yes |
| **Idempotent?** | ❌ No (fails if already exists) | ✅ Yes (create or update) |

> **💡 Rule of thumb:** Use imperative for learning/experiments. Use declarative (YAML files) for EVERYTHING in production!

---

## ⚡ Quick Reference Cheat Sheet

```bash
# THE DAILY COMMANDS
kubectl get pods -A -o wide               # See everything
kubectl describe pod <name>               # Debug a pod
kubectl logs <name> -f                    # Live logs
kubectl exec -it <name> -- sh             # Jump in
kubectl apply -f file.yaml                # Deploy
kubectl rollout undo deployment/<name>    # Rollback
kubectl port-forward svc/<name> 8080:80   # Local test
kubectl top pods                           # Resource usage
```

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between kubectl get and kubectl describe?**
> "kubectl get gives a brief status summary of resources (like a table view). kubectl describe gives full details including spec, status conditions, and Events. Events are especially useful for debugging — they show exactly what Kubernetes was doing and any errors."

**Q: Why use kubectl apply instead of kubectl create?**
> "kubectl apply is idempotent — it creates the resource if it doesn't exist, or updates it if it does. kubectl create fails if the resource already exists. apply is used in production and GitOps workflows because you can safely run it multiple times."

**Q: What does kubectl exec do?**
> "kubectl exec runs a command inside a running container. With -it, it opens an interactive terminal — basically an SSH-like shell into the container. It's essential for debugging — you can check environment variables, test network calls, or inspect files."

---

> **Ravi, you now have the full kubectl toolkit! 🎉** Practice these commands until they're muscle memory. Next → YAML manifests — understanding the language of Kubernetes! [11-yaml-manifest-files.md](11-yaml-manifest-files.md) 🚀
