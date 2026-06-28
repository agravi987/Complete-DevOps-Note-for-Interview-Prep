# 🧾 ConfigMaps — Configuration Separated from Code

> **Ravi! 👋** Imagine baking your app settings DIRECTLY into your Docker image. Every time you change an env variable, you rebuild and redeploy. 😩 Awful! **ConfigMaps** solve this — your config lives outside the image, and you change it without rebuilding anything. Smart! 🧠

---

## 🧠 The Problem ConfigMaps Solve

```
❌ The Old Way (Bad):
   Image → baked in APP_ENV=production, LOG_LEVEL=debug
   Need to change LOG_LEVEL? → Rebuild image → Redeploy!
   Wasted time. Different images for different envs. 😱

✅ The ConfigMap Way:
   Image → just the app code, no config
   ConfigMap → APP_ENV=production, LOG_LEVEL=debug (lives in cluster)
   Need to change LOG_LEVEL? → Update ConfigMap → Restart pod!
   Same image. Multiple environments. 🎉
```

---

## 📓 The Sticky Note Analogy

```
🏢 Your App = A new employee who doesn't know the office rules
📝 ConfigMap = The sticky note on their desk with all settings

"When you start, read the sticky note. It tells you:
 - Which database to connect to
 - What log level to use
 - Which feature flags are enabled"

If the rules change → update the sticky note (ConfigMap)
No need to hire a new employee (rebuild the image)!
```

---

## 📄 ConfigMap YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: prod

data:                          # Plain text key-value pairs
  APP_ENV: "production"
  LOG_LEVEL: "info"
  DB_HOST: "postgres.prod.svc.cluster.local"
  DB_PORT: "5432"
  MAX_CONNECTIONS: "100"

  # You can even store multi-line config files!
  nginx.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://backend:8080;
      }
    }
```

---

## 🔗 How to Use ConfigMap in a Pod

### Method 1: As Environment Variables (Most Common)

```yaml
spec:
  containers:
  - name: app
    image: my-app:v2
    env:
    # Inject a specific key
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config      # ConfigMap name
          key: APP_ENV          # Which key to inject

    # OR inject ALL keys at once!
    envFrom:
    - configMapRef:
        name: app-config        # All keys become env vars!
```

### Method 2: As a Mounted File (For config files)

```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d/  # Where to mount

  volumes:
  - name: config-volume
    configMap:
      name: app-config             # ConfigMap to mount
      items:
      - key: nginx.conf
        path: nginx.conf           # Filename inside mountPath
```

> 💡 **Ravi's Choice:** Use `env` method for simple settings. Use volume mount when you need an actual config file (nginx.conf, app.properties, etc.)

---

## 🎮 ConfigMap Commands

```bash
# List ConfigMaps
kubectl get configmap
kubectl get cm                       # Short form

# See the contents
kubectl describe configmap app-config

# View as YAML (see actual data!)
kubectl get configmap app-config -o yaml

# Create quickly from command line
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info

# Create from a .env file
kubectl create configmap app-config --from-env-file=.env

# Create from a whole file
kubectl create configmap nginx-config --from-file=nginx.conf

# Verify env vars inside a running pod
kubectl exec -it my-pod -- env | grep APP_ENV
kubectl exec -it my-pod -- env | grep LOG
```

---

## ⚠️ ConfigMap Update Behavior — Important!

| How Pod Uses ConfigMap | Updates Automatically? |
|---|---|
| `envFrom:` (env variables) | ❌ No — must restart pod |
| Volume mount | ✅ Yes — updates within ~60 seconds |

> 💡 **For env var changes:** `kubectl rollout restart deployment/my-app` — this restarts pods so they pick up new env vars!

---

## 🆚 ConfigMap vs Secret

| | ConfigMap 📝 | Secret 🔐 |
|---|---|---|
| **For** | Non-sensitive config | Sensitive data (passwords, tokens) |
| **Stored as** | Plain text | Base64 encoded |
| **View with kubectl** | Easy to read | Encoded (need to decode) |
| **Examples** | Log level, host URLs, feature flags | DB password, API keys, certs |
| **Encrypt at rest?** | No | Optional (with KMS) |

> 🚨 **Never put passwords or API keys in a ConfigMap! Use Secrets instead.**

---

## 🎤 Interview Knockout Answers

**Q: What problem does a ConfigMap solve?**
> "ConfigMaps decouple configuration from the application image. Instead of baking settings into the Docker image, they're stored in Kubernetes and injected at runtime. This allows the same image to run in multiple environments with different configurations, without rebuilding."

**Q: How does a Pod read a ConfigMap?**
> "Two ways: as environment variables (using envFrom or env.valueFrom.configMapKeyRef), or as mounted files in a volume. Env vars are simpler; volume mounts are better for config files like nginx.conf or application.properties."

**Q: Why not put config directly in the image?**
> "It couples your app image to a specific environment, forces a rebuild for every config change, leaks environment-specific details into the image, and makes the image non-portable across environments."

---

## ⚡ 30-Second Revision

```
🧾 ConfigMap = key-value store for non-sensitive config
📦 Separates config from image → no rebuild needed!
🔗 Inject as: env vars (envFrom) OR volume mount (as file)
⚠️  Env var changes need pod restart to take effect
🔄 Volume mounts auto-update (within ~60 seconds)
❌ NEVER put passwords/secrets in ConfigMap
🎮 kubectl get cm / kubectl describe cm / kubectl create cm
```

---

> **Ravi, one more to go! 🎉** ConfigMaps are for non-sensitive stuff. But passwords? DB credentials? That needs... **Secrets!** [13-secrets.md](13-secrets.md) 🚀
