# 🔐 Secrets — The Locked Safe of Kubernetes

> **Ravi! 👋** ConfigMaps are great for normal config. But passwords? API keys? Database credentials? Those need to be treated differently — they need **Secrets.** Think of it as the difference between writing a note on the whiteboard vs locking something in a safe. 🗝️💎

---

## 🧠 What is a Secret?

> **A Secret is a Kubernetes object that stores sensitive data like passwords, API keys, tokens, and certificates — kept separate from your app code.**

**The 3 reasons Secrets exist:**

1. 🔒 Keep sensitive data out of your YAML and source code
2. 🔑 Control access — only certain pods/roles get to read them
3. 🔄 Change credentials without rebuilding images

---

## 🗝️ The Locked Safe Analogy

```
📋 ConfigMap = Sticky note on the desk
   (Anyone walking by can read it)
   
🔐 Secret = Locked safe in the manager's office
   (Only authorized people have the key)
   
The data inside the safe (Secret) is:
  - Still stored in the cluster (etcd)
  - Base64 encoded (NOT encrypted by default!)
  - Only accessible to pods that are given permission
  - Access can be controlled via RBAC
```

> 🚨 **CRITICAL RAVI NOTE:** Base64 is **encoding**, NOT **encryption.** Anyone with access to the cluster can decode it. For true encryption, you need **etcd encryption at rest** or external tools like HashiCorp Vault / Sealed Secrets!

---

## 📄 Secret YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: prod

type: Opaque          # Generic secret type (most common)

# Option 1: stringData (plain text - Kubernetes auto-encodes)
stringData:
  DB_USER: "admin"
  DB_PASSWORD: "SuperSecurePassword123!"
  API_KEY: "sk-abc123xyz789"
```

```yaml
# Option 2: data (you pre-encode in base64)
data:
  DB_USER: YWRtaW4=          # echo -n "admin" | base64
  DB_PASSWORD: U3VwZXJTZWN1cmVQYXNzd29yZDEyMyE=
```

> 💡 **Use `stringData`** — it's cleaner and Kubernetes handles the encoding automatically!

---

## 🗂️ Secret Types

| Type | Use Case |
|---|---|
| `Opaque` | Generic key-value secrets (most common) |
| `kubernetes.io/service-account-token` | Service account auth tokens |
| `kubernetes.io/dockerconfigjson` | Docker registry auth |
| `kubernetes.io/tls` | TLS certificates and keys |
| `kubernetes.io/ssh-auth` | SSH private keys |

---

## 🔗 How to Use Secrets in a Pod

### Method 1: As Environment Variables

```yaml
spec:
  containers:
  - name: app
    image: my-app:v2
    env:
    # Inject specific key
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret       # Secret name
          key: DB_PASSWORD      # Which key

    # Inject ALL keys as env vars
    envFrom:
    - secretRef:
        name: db-secret
```

### Method 2: As Mounted Files (More Secure!)

```yaml
spec:
  containers:
  - name: app
    image: my-app:v2
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets   # Files appear here
      readOnly: true            # Always use readOnly!

  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
      # Each key becomes a file:
      # /etc/secrets/DB_USER
      # /etc/secrets/DB_PASSWORD
```

> 💡 **Ravi's Opinion:** Volume mount is more secure than env vars. Env vars can accidentally appear in logs. Files are harder to leak. For production, prefer volume mounts for sensitive secrets!

---

## 🎮 Secret Commands

```bash
# List secrets (data is hidden)
kubectl get secrets
kubectl get secret db-secret

# See Secret details (but NOT the actual values)
kubectl describe secret db-secret

# See the base64-encoded values
kubectl get secret db-secret -o yaml

# DECODE a base64 value (Linux/Mac)
echo "YWRtaW4=" | base64 -d

# Create a secret quickly
kubectl create secret generic db-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=SuperSecure123

# Create from a file (e.g., TLS certificate)
kubectl create secret tls my-tls-secret \
  --cert=tls.crt \
  --key=tls.key

# Create Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.company.com \
  --docker-username=ravi \
  --docker-password=mypassword

# Verify secret is injected into pod
kubectl exec -it my-pod -- env | grep DB_PASSWORD
kubectl exec -it my-pod -- cat /etc/secrets/DB_PASSWORD
```

---

## 🔒 Production Security Best Practices

```
1. Enable etcd encryption at rest
   → Encrypts secrets at the storage level
   
2. Use RBAC to restrict Secret access
   → Only pods that NEED a secret should GET it
   
3. Use External Secret Managers (Production Standard!)
   → HashiCorp Vault
   → AWS Secrets Manager
   → GCP Secret Manager
   → Azure Key Vault
   Operator pulls secrets dynamically, never stored in plain YAML
   
4. Never commit Secrets YAML to Git!
   → .gitignore your secrets files
   → Use Sealed Secrets or External Secrets Operator
   
5. Rotate secrets regularly
   → Update Secret → Restart pods to pick up new values
```

---

## 🆚 Secret vs ConfigMap

| | Secret 🔐 | ConfigMap 📝 |
|---|---|---|
| **For** | Passwords, tokens, certs | URLs, log levels, feature flags |
| **Encoding** | Base64 (data) or plain (stringData) | Plain text |
| **Visible in** | Encoded (requires decode) | Readable directly |
| **Access control** | Should be RBAC restricted | Less critical |
| **Examples** | `DB_PASSWORD`, `API_KEY`, TLS cert | `DB_HOST`, `LOG_LEVEL`, `APP_MODE` |

---

## 🎤 Interview Knockout Answers

**Q: What is a Secret in Kubernetes?**
> "A Secret is a Kubernetes object for storing sensitive data like passwords, API keys, and certificates. It's separate from ConfigMaps (which store non-sensitive config) and provides base64 encoding plus access control via RBAC."

**Q: Is base64 the same as encryption?**
> "No! Base64 is encoding, not encryption. Anyone who can access the Secret can decode it with a simple command. For real encryption, you need etcd encryption at rest or external secret management tools like HashiCorp Vault."

**Q: Why should Secrets be more protected than ConfigMaps?**
> "Secrets contain credentials that if leaked can lead to data breaches. They should be protected via RBAC (limiting who can read them), etcd encryption (encrypting them at storage), and ideally external secret managers that never store secrets in Kubernetes at all."

---

## ⚡ 30-Second Revision

```
🔐 Secret = Kubernetes object for sensitive data
🔑 Types: Opaque (generic), TLS, docker-registry, ssh
📦 Use stringData (plain) → K8s auto-encodes to base64
🔗 Inject as: env vars (secretKeyRef) OR volume mount
⚠️  Base64 ≠ Encryption (it's just encoding!)
🛡️  For real security: etcd encryption + RBAC + Vault
🚫 NEVER commit Secret YAML to Git!
```

---

## 🎉 Phase 1 Complete — You Did It, Ravi!

```
✅ Kubernetes Architecture   — Big picture: control plane + nodes
✅ Components               — Every player and their job
✅ Pods                     — The atomic unit
✅ Multi-Container Pods     — Sidecar patterns
✅ ReplicaSets              — Keep pods alive
✅ Deployments              — Rolling updates + rollback
✅ Namespaces               — Organize your cluster
✅ Labels & Selectors       — The glue connecting everything
✅ Services                 — Stable network access to pods
✅ kubectl Commands         — Your daily toolkit
✅ YAML Manifests           — The language of Kubernetes
✅ ConfigMaps               — Externalize non-sensitive config
✅ Secrets                  — Secure sensitive data
```

> **You're Phase 1 certified, Ravi! 🚀🎓** You now understand the foundation of Kubernetes. Real clusters run on exactly these concepts. Go to Phase 2 when you're comfortable — don't rush, revise these first! 💪

**Back to:** [Phase 1 README](README.md) | **Next Phase:** [Phase 2 - Production Essentials](../phase-2-production-essentials/README.md) 🚀
