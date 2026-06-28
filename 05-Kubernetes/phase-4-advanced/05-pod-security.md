# 🛡️ Pod Security Standards & SecurityContext

> **Hey Ravi! 👋** Did you know that by default, a container in Kubernetes runs as the `root` user? If a hacker breaks into your container, they have `root` access! 😱 Worse, if they escape the container, they have `root` access to the actual physical server! We need to lock these containers in a cage. Let's talk about **Pod Security**. 🔒

---

## 🧠 What is a SecurityContext?

> **A SecurityContext defines privilege and access control settings for a Pod or Container. It's how you tell Linux: "Do NOT let this app run as root, and DO NOT let it write to the hard drive!"**

You can apply it at the **Pod level** (applies to all containers inside) or the **Container level**.

---

## 🛑 The 3 Pod Security Standards (PSS)

Kubernetes officially defines 3 levels of strictness for clusters:

1. **Privileged (The Wild West):** 🤠 Unrestricted. Used for system agents (like CNI plugins or logging agents that *need* root access to the node).
2. **Baseline (The Middle Ground):** 😐 Prevents known privilege escalations, but still allows some flexibility. Good default for legacy apps.
3. **Restricted (Fort Knox):** 🏰 Highly restricted. Enforces strict best practices (must run as non-root). This is what you want for production web apps!

---

## 📄 Real YAML (The "Restricted" Container)

This is what a perfectly secure, locked-down container looks like. (Memorize these 4 fields for interviews!)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:         # POD LEVEL
    runAsUser: 1000        # Run as user ID 1000 (NOT 0/root!)
    runAsGroup: 3000
    fsGroup: 2000          # Group ID that owns mounted volumes

  containers:
  - name: app
    image: my-app:v1
    securityContext:       # CONTAINER LEVEL
      # 1. Prevent the process from gaining more privileges than it started with
      allowPrivilegeEscalation: false
      
      # 2. Make the entire container filesystem READ-ONLY! 
      # (Hackers can't download malware!)
      readOnlyRootFilesystem: true
      
      # 3. Strip all Linux kernel capabilities
      capabilities:
        drop:
        - ALL
```
*Note: If `readOnlyRootFilesystem` is true, but your app needs to write temp files, you must mount an `emptyDir` volume to `/tmp`!*

---

## 🛡️ Enforcing Standards (Pod Security Admission)

Writing the YAML above is great, but developers are lazy. They will forget to add it. You need to FORCE them to do it.

Using the **Pod Security Admission** controller (built-in!), you apply a label to a Namespace.

```bash
# Force every pod in the 'prod' namespace to meet the 'Restricted' standard!
kubectl label namespace prod pod-security.kubernetes.io/enforce=restricted

# (Optional) Just AUDIT the 'dev' namespace (don't block, just log warnings)
kubectl label namespace dev pod-security.kubernetes.io/audit=restricted
```
If a developer tries to deploy a pod running as `root` into `prod`, the API server will reject it! 🚫

---

## 🐧 AppArmor & Seccomp (Linux Kernel Magic)

For extreme security, K8s can hook directly into the Linux kernel!

- **Seccomp (Secure Computing):** Restricts which *system calls* the container can make to the Linux kernel (e.g., "You can read files, but you cannot change the system time").
- **AppArmor:** A Linux kernel security module that restricts programs' capabilities with per-program profiles.

*(Usually, applying the `Restricted` PSS automatically enables the default Seccomp profile!)*

---

## 🎤 Interview Knockout Answers

**Q: What is a SecurityContext in Kubernetes?**
> "A SecurityContext is a set of properties defined in a Pod or Container spec that controls its security permissions and privileges. It allows you to enforce that containers run as a non-root user, make the root filesystem read-only, drop Linux kernel capabilities, and prevent privilege escalation."

**Q: Why is running a container as root dangerous?**
> "By default, Docker and Kubernetes run containers as the `root` user (UID 0). If a vulnerability allows an attacker to break out of the container (container escape), they will have root-level access on the underlying worker node, allowing them to compromise the entire cluster."

**Q: How do you enforce Pod Security Standards across a cluster?**
> "You use the built-in Pod Security Admission controller. By applying specific labels to Namespaces (like `pod-security.kubernetes.io/enforce=restricted`), you instruct the Kubernetes API server to reject any Pod deployments into that namespace that do not adhere to the highly restricted security profile."

---

## ⚡ 30-Second Revision

- 🔒 **SecurityContext** = Limits container privileges.
- 🧑‍💻 `runAsUser: 1000` = NEVER run as `root` (UID 0)!
- 🚫 `allowPrivilegeEscalation: false` = Stop hackers from gaining `sudo`.
- 📁 `readOnlyRootFilesystem: true` = Stop malware from being downloaded.
- 🛡️ **Pod Security Standards (PSS):** Privileged, Baseline, Restricted.
- 🛑 Use **Namespace Labels** to enforce these standards across the cluster.

> Next → [Advanced Storage (CSI)](06-storage-advanced.md) 🚀
