# 🛠️ Kustomize — The Configuration Customizer

> **Hey Ravi! 👋** Remember Helm? Helm is great for packaging apps with `{{ variables }}` to share with the world. But what if you just have your OWN company's app, and you want to deploy it to `dev`, `staging`, and `prod`? Do you really want to write a complex Helm chart just to change the DB password and replica count across environments? No! You use **Kustomize**. 🧩

---

## 🧠 What is Kustomize?

> **Kustomize is a configuration management tool built directly into `kubectl`. It lets you customize raw YAML files for different environments (overlays) without using templates or variables.**

It uses a "Base and Overlay" model.

---

## 🍔 The Burger Analogy

- **The Base (The Standard Burger):** A plain burger with a bun and a patty. This is your core `deployment.yaml` and `service.yaml`. It represents your app.
- **The Overlays (The Toppings):** 
  - **Dev Overlay:** Adds ketchup and pickles. (Adds a debug flag, sets `replicas: 1`).
  - **Prod Overlay:** Adds double bacon, cheese, and a fancy box. (Adds a production DB Secret, sets `replicas: 10`).

Kustomize takes the Base, slaps the Overlay on top of it, and gives you the final customized YAML! 🍔✨

---

## 📁 File Structure

```text
my-app/
├── base/
│   ├── deployment.yaml      # Standard app
│   ├── service.yaml         # Standard service
│   └── kustomization.yaml   # Points to the above files
│
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml   # Points to base + adds dev patches
    │   └── patch-replicas.yaml  # Replicas = 1
    │
    └── prod/
        ├── kustomization.yaml   # Points to base + adds prod patches
        └── patch-replicas.yaml  # Replicas = 10
```

---

## 📄 Real YAML (The Magic `kustomization.yaml`)

Let's look at `overlays/prod/kustomization.yaml`. This is where the magic happens!

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 1. Start with the Base!
resources:
- ../../base

# 2. Add a prefix to all object names! (e.g., 'web' becomes 'prod-web')
namePrefix: prod-

# 3. Add a label to EVERY single resource automatically!
commonLabels:
  environment: production

# 4. Patch the Base Deployment to have 10 replicas!
patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: web        # Matches the base name
    spec:
      replicas: 10     # OVERWRITES the base replica count!
```

---

## 🎮 Commands (It's built into kubectl!)

You don't need to install anything! Kustomize is natively supported by `kubectl` using the `-k` flag!

```bash
# Preview what the final YAML will look like for PROD
kubectl kustomize ./overlays/prod

# Apply the PROD overlay directly to the cluster!
kubectl apply -k ./overlays/prod

# Apply the DEV overlay
kubectl apply -k ./overlays/dev
```

---

## 🆚 Kustomize vs Helm

This is a classic interview question! Which one should you use?

| Feature | Kustomize 🧩 | Helm 📦 |
|---|---|---|
| **Style** | Overlays and Patches (Patching) | Templating (`{{ .Values }}`) |
| **Best For** | Internal company apps across envs (Dev/Prod). | Distributing public apps (like Redis/MySQL). |
| **Complexity** | Low (It's just pure YAML). | High (Go templating can get messy). |
| **Installation** | Built into `kubectl`! | Requires installing the `helm` CLI. |
| **Lifecycle** | Just generates YAML (stateless). | Manages Releases, tracking history (stateful). |

---

## 🎤 Interview Knockout Answers

**Q: What is Kustomize and how does it differ from Helm?**
> "Kustomize is a configuration management tool built into `kubectl` that uses a template-free, 'base and overlay' approach to patch YAML files for different environments. Helm is a package manager that uses Go templating to inject variables into YAML. Kustomize is generally better for internal, multi-environment app deployments, while Helm is better for packaging and distributing complex third-party applications."

**Q: How do you apply a Kustomize configuration to a cluster?**
> "Because Kustomize is integrated into `kubectl`, you can simply run `kubectl apply -k <directory>`, where the directory contains a `kustomization.yaml` file. You don't need any external tools."

**Q: What is a `kustomization.yaml` file used for?**
> "It is the instruction manual for Kustomize. It defines which raw YAML files to include as `resources`, and what transformations to apply to them, such as adding `namePrefixes`, injecting `commonLabels`, generating ConfigMaps, or applying YAML `patches` to overwrite specific fields."

---

## ⚡ 30-Second Revision

- 🧩 **Kustomize** = YAML patching (Base + Overlays).
- 🍔 **Base** = Standard YAML. **Overlay** = Environment-specific tweaks (Dev/Prod).
- 🚫 **No Templates:** You don't use `{{ variables }}`. It's all pure YAML.
- 🏷️ Great for adding a `commonLabel` to 50 files at once!
- 🛠️ Natively built into kubectl using `kubectl apply -k`.
- 🆚 Use Kustomize for *your* apps, use Helm for *other people's* apps (like Prometheus).

> Next → [Custom Resource Definitions (CRDs)](02-custom-resource-definitions.md) 🚀
