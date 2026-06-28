# 🌐 Services — The Stable Front Door for Your Pods

> **Ravi! 👋** Here's a problem: Pods die and come back with NEW IP addresses. Your other apps can't rely on Pod IPs because they keep changing. **Services** solve this — they give a stable, permanent address in front of your ever-changing Pods. 🚪✨

---

## 🧠 The Problem Services Solve

```
Without Service:
  App A needs to call App B
  App B Pod IP: 10.0.0.5

  Pod B crashes → new Pod B IP: 10.0.0.12
  App A is still calling 10.0.0.5 → ❌ BROKEN!

With Service:
  App A calls Service "app-b" (stable name, stable virtual IP)
  Service → always finds current Pod B (by label!)
  Pod dies? Service finds new pod automatically → ✅ WORKS!
```

---

## 🛎️ The Hotel Front Desk Analogy

```
🏨 Hotel = Kubernetes cluster
🛎️ Front Desk (Service) = stable contact point
🏠 Rooms (Pods) = they change, get renovated, move around
👤 Guest (client) = other apps or users

Guest says: "I want to reach Room Service (stable name)"
Front Desk: "Sure! Let me find which pod is available right now"
→ Routes to a healthy pod automatically!

If a room is under maintenance (pod died) → desk routes to another room
Guest never needs to know the room number! 🎯
```

---

## 🔧 How Services Work Internally

```
1. You create a Service with a selector (e.g., app=web)
2. Kubernetes creates a stable virtual IP (ClusterIP) for the Service
3. kube-proxy maintains iptables/IPVS rules on every node
4. Kubernetes creates an Endpoints object that lists current Pod IPs

When traffic hits the Service IP:
  → kube-proxy routes it to one of the Pod IPs (round-robin)
  → If a pod dies, Endpoints updates automatically
  → Traffic always reaches healthy pods!
```

---

## 🗂️ The 4 Service Types — Know All 4!

### Type 1: ClusterIP (Default) 🏠
```
Access: ONLY within the cluster
Use for: Internal microservice communication
```
```yaml
type: ClusterIP
# Creates a virtual IP accessible only inside the cluster
# cluster.local DNS: <service-name>.<namespace>.svc.cluster.local
```

### Type 2: NodePort 🚪
```
Access: External via NodeIP:NodePort
Use for: Development, testing, simple setups
Port range: 30000–32767
```
```yaml
type: NodePort
# Opens a port on EVERY node in the cluster
# Access: http://<any-node-ip>:30080
```

### Type 3: LoadBalancer ☁️
```
Access: External via cloud load balancer IP
Use for: Production external traffic (AWS ELB, GCP LB)
```
```yaml
type: LoadBalancer
# Cloud provisions a load balancer automatically
# Most common for public-facing apps in cloud
```

### Type 4: ExternalName 🔗
```
Access: Maps Service to an external DNS name
Use for: Connecting to external databases, APIs
```
```yaml
type: ExternalName
externalName: my-database.external.com
# No proxying — just DNS CNAME aliasing
```

---

## 📄 Service YAML — Full Explained

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service          # Service name (becomes DNS name!)
  namespace: prod

spec:
  type: ClusterIP            # Service type

  selector:
    app: web-frontend        # Route traffic to pods with this label

  ports:
  - name: http
    port: 80                 # Port the SERVICE listens on
    targetPort: 8080         # Port the CONTAINER listens on
    protocol: TCP
```

> 💡 **Ravi's Tip:** `port` = what clients hit | `targetPort` = what the pod actually listens on. Clients call port 80, pod receives on 8080!

---

## 📄 NodePort YAML Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web-frontend
  ports:
  - port: 80           # ClusterIP port (internal)
    targetPort: 8080   # Container port
    nodePort: 30080    # External port on each node (30000-32767)
```

---

## 🎮 Service Commands

```bash
# List services
kubectl get svc
kubectl get services

# Detailed service info (endpoints, selector, ports)
kubectl describe svc web-service

# See which pods the service is targeting
kubectl get endpoints web-service

# Test service from inside cluster (run a debug pod)
kubectl run test --image=busybox --rm -it -- sh
# Inside the test pod:
wget -O- web-service:80

# Port forward for local testing (no NodePort needed!)
kubectl port-forward svc/web-service 8080:80
# Now access http://localhost:8080 locally
```

---

## 🐛 Debugging Services — The Checklist

```
Service traffic not working? Go through this list:

Step 1: kubectl get svc               → Service exists? IP assigned?
Step 2: kubectl get endpoints <svc>   → Are there pods in endpoints?
         If no endpoints → Label mismatch! (selector != pod label)
Step 3: kubectl get pods -l app=web   → Pods running?
Step 4: kubectl describe svc <name>   → Any events?
Step 5: Test from inside cluster with curl/wget
```

> 💡 **The #1 debugging tip:** `kubectl get endpoints`. If endpoints are empty → your Service selector doesn't match any pod labels!

---

## 🆚 Service Types Comparison

| Type | Accessible From | Use Case | Cost |
|---|---|---|---|
| `ClusterIP` | Inside cluster only | Microservice comms | Free |
| `NodePort` | Outside (via node IP) | Dev/testing | Free |
| `LoadBalancer` | Outside (via LB IP) | Production external | 💰 Cloud charges |
| `ExternalName` | Inside cluster | Map to external service | Free |

---

## 🎤 Interview Knockout Answers

**Q: Why do Kubernetes apps need Services?**
> "Pod IPs are ephemeral — they change every time a pod is created. Services provide a stable virtual IP and DNS name that routes traffic to the right pods dynamically using label selectors. Without Services, apps couldn't reliably communicate with each other."

**Q: What is the difference between port and targetPort?**
> "port is the port the Service itself listens on — what clients use to call the Service. targetPort is the port the container inside the Pod listens on. They can be different — Service decouples the port clients know from the port the app uses."

**Q: Why is ClusterIP the default Service type?**
> "Most inter-service communication is internal. ClusterIP is the simplest, most secure option — only accessible within the cluster. External access should be handled by LoadBalancer or Ingress, not by exposing every service directly."

**Q: What is an Endpoint object?**
> "An Endpoints object is automatically created for each Service. It contains the list of current Pod IPs that match the Service selector. It updates dynamically as pods come and go. If Endpoints is empty, no traffic will reach pods."

---

## ⚡ 30-Second Revision

```
🌐 Service = stable virtual IP + DNS in front of pods
🏷️  Finds pods using label selectors (dynamic!)
📋 4 types: ClusterIP / NodePort / LoadBalancer / ExternalName
🔑 port (service) vs targetPort (container)
🐛 Debug: kubectl get endpoints (if empty → label mismatch!)
🌍 For external traffic in prod → use LoadBalancer or Ingress
🔗 DNS: <service>.<namespace>.svc.cluster.local
```

---

> **Ravi, Services are the networking backbone! 🎉** Your apps need to talk to each other — Services make that reliable. Next → Master `kubectl` commands, your main tool! [10-kubectl-commands.md](10-kubectl-commands.md) 🚀
