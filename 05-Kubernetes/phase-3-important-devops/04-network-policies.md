# 🧱 Network Policies — The Kubernetes Firewall

> **Hey Ravi! 👋** Did you know that by default in Kubernetes, **EVERY pod can talk to EVERY other pod**? Even across different namespaces! If a hacker compromises your frontend web pod, they can just casually ping your backend database pod. Yikes! 😱 We fix this gaping security hole using **Network Policies**. 🧱

---

## 🧠 What is a Network Policy?

> **A NetworkPolicy is a firewall rule for your Pods. It controls which Pods can receive traffic (Ingress) and which external endpoints they can send traffic to (Egress).**

⚠️ **CRITICAL REQUIREMENT:** Network Policies DO NOTHING unless you have a Network Plugin (CNI) that supports them! (Calico, Cilium, Weave support them. Flannel does NOT).

---

## 🏰 The Castle Analogy

- **Default K8s:** A flat field where anyone can walk up to anyone else and talk.
- **Network Policy:** Building a castle wall around a specific Pod.
  - **Ingress Rule:** "Only the knights (Frontend Pods) can enter the castle gates."
  - **Egress Rule:** "The people inside the castle can only send letters to the King (External API), nowhere else."

---

## 🛡️ The "Default Deny All" Pattern (Best Practice!)

Since everything is open by default, the first thing security engineers do is create a policy that blocks EVERYTHING in a namespace. 

```yaml
# 🚨 Blocks ALL incoming and outgoing traffic in the 'prod' namespace!
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: prod
spec:
  podSelector: {}    # Empty selector matches ALL pods in namespace
  policyTypes:
  - Ingress
  - Egress
```
Once this is applied, all apps will break! 💥 Then, you explicitly write rules to allow *only* the traffic you want.

---

## 📄 Real YAML (Allow Frontend to talk to Backend)

Let's allow pods labeled `app=frontend` to talk to pods labeled `app=backend` on port 3306.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: prod
spec:
  # 1. WHO DOES THIS POLICY PROTECT? (The Castle)
  podSelector:
    matchLabels:
      app: backend      # Apply this wall around the backend pods
      
  policyTypes:
  - Ingress
  
  # 2. WHO IS ALLOWED IN? (The Knights)
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend # Only allow frontend pods to enter!
    ports:
    - protocol: TCP
      port: 3306        # Only on port 3306
```

---

## 🧩 The 3 Ways to Select Traffic (`from` or `to`)

Inside an `ingress: from:` or `egress: to:` block, you can allow traffic based on:

1. **`podSelector`**: Allow specific Pods in the *same* namespace.
2. **`namespaceSelector`**: Allow *all* Pods from a specific Namespace (e.g., allow everything in `monitoring` namespace to scrape metrics).
3. **`ipBlock`**: Allow traffic to/from external IP ranges outside the cluster (e.g., allow traffic only from your corporate VPN IP `192.168.1.0/24`).

---

## 🎮 Commands

```bash
# List network policies in current namespace
kubectl get networkpolicy
kubectl get netpol    # Short form!

# See the rules clearly formatted
kubectl describe netpol allow-frontend-to-backend

# Test connectivity! (Exec into frontend, try to hit backend)
kubectl exec -it frontend-pod -- curl -m 3 http://backend-svc:8080
```

---

## 🎤 Interview Knockout Answers

**Q: Are Pods isolated by default in Kubernetes?**
> "No, by default Kubernetes operates a flat network where all Pods can communicate with all other Pods across all namespaces. Isolation must be explicitly configured using NetworkPolicies."

**Q: If I create a NetworkPolicy, but it doesn't seem to be blocking traffic, what is the most likely cause?**
> "The most common reason is that the cluster's Container Network Interface (CNI) plugin does not support Network Policies. For example, Flannel does not enforce them. You need a CNI like Calico, Cilium, or Weave Net for the rules to actually be enforced."

**Q: Explain the 'Default Deny' pattern.**
> "The Default Deny pattern is a security best practice where you create a NetworkPolicy with an empty `podSelector` (matching all pods) and no `ingress` or `egress` rules. This immediately blocks all traffic in the namespace. You then create additional, specific NetworkPolicies to explicitly whitelist the required traffic flows (Principle of Least Privilege)."

---

## ⚡ 30-Second Revision

- 🧱 **NetworkPolicy** = Internal firewall for Pods.
- 🔓 Default state = All traffic allowed.
- 🛑 **Ingress** = Incoming traffic rules.
- 📤 **Egress** = Outgoing traffic rules.
- 🛡️ Best practice = Create a "Default Deny All" policy, then whitelist explicitly.
- ⚠️ Requires a compatible CNI (Calico/Cilium) to actually work!

> Next → [Helm](05-helm.md) 🚀
