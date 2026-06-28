# 🛂 Admission Controllers — The K8s Bouncers

> **Hey Ravi! 👋** Let's say a developer tries to deploy a Pod without any memory limits. K8s will accept it, right? Yes. What if they try to pull an image from `docker.io` instead of your secure company registry? K8s accepts that too. If you want to STOP bad things from entering the cluster, you need **Admission Controllers**. They are the bouncers at the door of the API server! 🛑

---

## 🧠 What is an Admission Controller?

> **An Admission Controller is a piece of code that intercepts requests to the Kubernetes API server AFTER the request is authenticated/authorized, but BEFORE the object is saved to the `etcd` database.**

---

## 🏛️ The API Server Flow (The Security Checkpoint)

When you run `kubectl apply -f pod.yaml`, here is exactly what happens:

1. **Authentication:** "Who are you?" (Are you Ravi?)
2. **Authorization (RBAC):** "Are you allowed to create Pods?"
3. **Mutating Admission:** "Let me silently modify your Pod YAML before saving it." (e.g., injecting a sidecar). 🪄
4. **Validating Admission:** "Let me check if your YAML follows company rules. If not, DENIED!" 🛑
5. **Storage:** Saves to `etcd` database.

---

## 🪄 1. Mutating Webhooks (The Helpful Bouncer)

Mutating controllers **modify** the YAML on the fly. 

**Real World Use Cases:**
- **Istio Service Mesh:** When you deploy a Pod, the Istio Mutating Webhook intercepts it and silently injects a second container (the Envoy Proxy sidecar) into your Pod! You didn't write it in your YAML, K8s added it for you.
- **Defaulting:** Automatically adding a default `StorageClass` to a PVC if the user forgot to specify one.

---

## 🛑 2. Validating Webhooks (The Strict Bouncer)

Validating controllers strictly **approve or deny** requests. They cannot modify.

**Real World Use Cases:**
- **Security:** "DENIED: Your container is trying to run as `root` user."
- **Governance:** "DENIED: Your Pod is missing the `billing-department` label."
- **Resource Limits:** "DENIED: You forgot to set CPU limits."

---

## 📜 OPA Gatekeeper (The Policy Engine)

Writing custom webhooks in Go/Python is hard. So the industry standard is to use **OPA Gatekeeper**. 

Gatekeeper is a Validating Webhook that lets you write company policies using a simple language called **Rego**. 

*Example Gatekeeper Policy:* "All containers must pull images from `registry.company.com`. Reject anything else."

---

## 🛡️ Pod Security Admission (PSA)

This is a built-in Kubernetes Admission Controller! (It replaced the deprecated PodSecurityPolicies).

You apply a simple label to a Namespace, and the PSA controller will block any Pods that violate security standards.

```bash
# Label the 'prod' namespace to enforce 'restricted' security!
kubectl label namespace prod pod-security.kubernetes.io/enforce=restricted
```
*(If a developer tries to deploy a privileged pod into `prod` now, the API server will reject it with a big error!)*

---

## 🎮 Commands

```bash
# See which admission webhooks are installed in your cluster!
kubectl get validatingwebhookconfigurations
kubectl get mutatingwebhookconfigurations
# (You might see gatekeeper, istio, or cert-manager in the list!)

# If your pod is mysteriously failing to create with an API error,
# an Admission Controller blocked it! Read the error message carefully.
```

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between a Mutating Admission Controller and a Validating Admission Controller?**
> "Both intercept requests to the API server before they are persisted to etcd. A Mutating controller runs first and can modify the incoming object—for example, injecting a sidecar container or setting default labels. A Validating controller runs second; it cannot modify the object, it can only inspect it and return an Approve or Deny response based on policy."

**Q: Where do Admission Controllers sit in the Kubernetes API request lifecycle?**
> "They sit between Authorization (RBAC) and Persistence (etcd). After a user is authenticated and authorized via RBAC, the request passes through Mutating admission, then schema validation, then Validating admission. Only if it survives all of these is it saved to etcd."

**Q: What is OPA Gatekeeper used for in Kubernetes?**
> "OPA Gatekeeper is an open-source policy engine that acts as a Validating Admission Webhook. Instead of writing custom Go code for every company rule, Gatekeeper allows platform engineers to define strict governance and security policies (like enforcing mandatory labels or restricting image registries) using a policy language called Rego."

---

## ⚡ 30-Second Revision

- 🛂 **Admission Controllers** intercept API requests *before* they are saved.
- 🪄 **Mutating** = Modifies YAML on the fly (e.g., injecting Istio sidecars).
- 🛑 **Validating** = Approves or rejects YAML based on rules.
- 📜 **OPA Gatekeeper** = The industry standard tool for writing validation policies.
- 🛡️ **Pod Security Admission** = Built-in controller to enforce restricted pod security.
- 🕵️‍♂️ If `kubectl apply` returns a weird error about policies, an Admission Controller blocked you!

> Next → [Pod Security Standards](05-pod-security.md) 🚀
