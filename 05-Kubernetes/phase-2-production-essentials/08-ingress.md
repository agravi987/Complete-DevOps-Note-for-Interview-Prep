# 🔀 Ingress — The Smart Traffic Cop

> **Hey Ravi! 👋** You know how a `LoadBalancer` Service gives your app a public IP? Great! But what if you have 20 microservices? Do you want to pay AWS for 20 expensive Load Balancers? 💸 No! You want ONE public entry point that routes traffic smartly based on URLs. Enter **Ingress**! 🚦

---

## 🧠 What is Ingress?

> **Ingress is a smart router. It exposes HTTP and HTTPS routes from outside the cluster to Services within the cluster based on rules (like hostnames or URL paths).**

⚠️ **Important:** Ingress is NOT a Service type (like NodePort or ClusterIP). It sits *in front* of your Services.

---

## 🏨 The Hotel Receptionist Analogy

- **LoadBalancer Service (The Door):** You have 5 restaurants in your hotel, and you build 5 separate front doors to the street. (Expensive and weird).
- **Ingress (The Receptionist):** You build ONE big, fancy front door. Guests walk in and ask the receptionist:
  - "I want `example.com/pizza`" → Receptionist sends them to Pizza Service.
  - "I want `sushi.example.com`" → Receptionist sends them to Sushi Service.
  - "Here is my SSL certificate" → Receptionist checks it and decrypts the traffic! 🔐

---

## ⚙️ The Missing Piece: Ingress Controller

Creating an Ingress YAML does absolutely **NOTHING** by itself. 
It's just a set of rules. You need a program running in the cluster to actually READ those rules and route the traffic.

That program is the **Ingress Controller**.
*(Popular ones: NGINX Ingress, Traefik, AWS ALB Controller).*

```text
Internet 
   ↓
Load Balancer (Cloud Provider)
   ↓
Ingress Controller (NGINX Pod) 🧠 Reads the Ingress Rules!
   ↓
Routes to ClusterIP Services
```

---

## 📄 Real YAML (Host and Path Based Routing)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-smart-router
  annotations:
    # Tells K8s which controller should handle this!
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx   # I want the NGINX controller!
  
  rules:
  # 1. HOST-BASED ROUTING (Subdomains)
  - host: api.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-api-svc
            port:
              number: 80

  # 2. PATH-BASED ROUTING (Folders)
  - host: mycompany.com
    http:
      paths:
      - path: /blog
        pathType: Prefix
        backend:
          service:
            name: blog-svc
            port: 
              number: 80
      - path: /store
        pathType: Prefix
        backend:
          service:
            name: store-svc
            port: 
              number: 80
```

---

## 🔐 TLS Termination (HTTPS)

Ingress is awesome because you can put your SSL certificate on the Ingress, and it handles decrypting traffic. Your backend apps just use plain HTTP (saving CPU)!

```yaml
spec:
  tls:
  - hosts:
      - mycompany.com
    secretName: my-tls-secret   # A K8s Secret containing cert & key!
  rules:
  # ... rules ...
```
*(Combined with `cert-manager`, K8s can even auto-renew Let's Encrypt certs for you!)*

---

## 🎮 Commands

```bash
# Get Ingress rules
kubectl get ingress
kubectl get ing      # Short form!

# Describe to see if it's routing correctly
kubectl describe ingress my-smart-router

# Check the actual Ingress Controller logs (If using NGINX)
# (This is where you look if you're getting 502 Bad Gateway!)
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between a LoadBalancer Service and Ingress?**
> "A LoadBalancer Service operates at Layer 4; it provisions a cloud load balancer for a single service and routes TCP/UDP traffic blindly. Ingress operates at Layer 7 (HTTP/HTTPS); it uses a single load balancer to route traffic to multiple internal Services based on hostnames or URL paths. This saves money and centralizes SSL termination."

**Q: Why do you need an Ingress Controller?**
> "An Ingress object is just a Kubernetes resource that stores routing rules. It doesn't actually route traffic. You need an Ingress Controller (like NGINX or Traefik) running in the cluster to actively read those rules and implement the actual proxying of traffic."

**Q: How does Ingress handle SSL/TLS?**
> "Ingress performs TLS termination. You store your SSL certificate and private key in a Kubernetes Secret, and reference that Secret in the Ingress YAML. The Ingress Controller decrypts the incoming HTTPS traffic and forwards it as plain HTTP to the backend pods."

---

## ⚡ 30-Second Revision

- 🚦 **Ingress** = Layer 7 smart router (HTTP/HTTPS).
- 🔀 Routes based on **Host** (`api.domain.com`) or **Path** (`domain.com/api`).
- 🧠 **Ingress Controller** = The actual engine (e.g., NGINX) that executes the rules.
- 💰 **Saves Money:** 1 Cloud LB for 50 apps, instead of 50 Cloud LBs.
- 🔐 **TLS Termination:** Handles HTTPS at the edge so your apps don't have to.
- 📝 Need a Secret to store the TLS certs.

> Next → [Horizontal Pod Autoscaler (HPA)](09-hpa.md) 🚀
