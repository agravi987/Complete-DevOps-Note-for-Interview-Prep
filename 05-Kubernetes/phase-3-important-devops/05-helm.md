# 📦 Helm — The Kubernetes Package Manager

> **Hey Ravi! 👋** Have you ever tried to install a complex app like Prometheus? It requires 15 different YAML files (Deployments, Services, Roles, ConfigMaps...). Imagine copying and pasting 15 files and manually changing the namespace in every single one. Gross! 🤮 Enter **Helm**, the `apt-get` or `npm` of Kubernetes! 🚢

---

## 🧠 What is Helm?

> **Helm is a package manager for Kubernetes. It packages multiple YAML files into a single bundle called a "Chart", allowing you to install, upgrade, and configure complex applications with one command.**

---

## 🛍️ The Ikea Furniture Analogy

- **Raw YAML (kubectl apply):** Buying a pile of wood, screws, and glue from the hardware store and figuring out how to build a chair from scratch. 🪵
- **Helm (helm install):** Buying a flat-pack chair from Ikea. It comes with instructions (Templates) and you just choose the color (Values). 🪑

---

## 🧩 The 3 Core Concepts of Helm

1. **Chart:** The package itself (the Ikea box). It contains all the YAML templates and default settings.
2. **Values (`values.yaml`):** The variables you can change! Want 3 replicas instead of 1? Want to change the image tag? You override the defaults here.
3. **Release:** An actual installed instance of a Chart in your cluster. (You can install the same WordPress chart 5 times to get 5 different releases/blogs!).

---

## 📄 How Helm Templating Works

Helm uses Go templating to inject variables into YAML files.

**The Template (`deployment.yaml` inside the Chart):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-web
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

**Your Overrides (`my-values.yaml`):**
```yaml
replicaCount: 5
image:
  repository: nginx
  tag: "1.28"
```

When you run `helm install`, Helm merges them together and sends the final, perfect YAML to Kubernetes! ✨

---

## 🎮 Helm Commands (Your Daily Drivers)

```bash
# 1. Add a Repository (Like an App Store)
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# 2. Search for an app
helm search repo wordpress

# 3. Install an app (Name it 'my-blog')
helm install my-blog bitnami/wordpress

# 4. Install WITH custom overrides
helm install my-blog bitnami/wordpress -f my-values.yaml

# 5. See what's installed in the cluster
helm list       # or helm ls

# 6. Upgrade to a new version or new values
helm upgrade my-blog bitnami/wordpress -f new-values.yaml

# 7. Uh oh, it broke! Rollback! ⏪
helm history my-blog
helm rollback my-blog 1   # Rollback to revision 1!

# 8. Delete the app (Cleans up ALL 15 YAML files instantly!)
helm uninstall my-blog
```

---

## 🆚 Helm vs Kustomize

*Note: We'll dive deeper into Kustomize in Phase 4!*

- **Helm:** Template engine. Uses `{{ variables }}`. Great for distributing apps to OTHER people (like open source charts).
- **Kustomize:** Overlay engine. Uses base YAML + patches. Great for your OWN company's internal apps across dev/staging/prod environments without writing `{{ variables }}` everywhere.

---

## 🎤 Interview Knockout Answers

**Q: What is Helm and why do we use it?**
> "Helm is the package manager for Kubernetes. We use it because deploying complex applications involves managing many interdependent YAML resources. Helm bundles these into a 'Chart', allows us to parameterize configurations using `values.yaml`, and manages the entire lifecycle of the application as a single 'Release', supporting easy upgrades and rollbacks."

**Q: Explain the relationship between a Chart, a Release, and Values.**
> "A Chart is the package containing the Kubernetes YAML templates. Values are the configuration settings (like replica counts or image tags) injected into those templates. A Release is a specific instance of a Chart deployed into the cluster with a specific set of Values."

**Q: If you deploy an application with `kubectl apply`, how does that differ from deploying with `helm install` when it comes to rollbacks?**
> "With raw `kubectl apply`, Kubernetes only tracks rollout history for Deployments/DaemonSets/StatefulSets. If a config change involved a ConfigMap or Secret, rolling back the Deployment won't revert those. Helm tracks the state of the ENTIRE release (all resources combined). A `helm rollback` restores the exact state of every single resource in that package."

---

## ⚡ 30-Second Revision

- 📦 **Helm** = Kubernetes Package Manager.
- 🗺️ **Chart** = The template package.
- ⚙️ **Values** = The variables you inject.
- 🚀 **Release** = The installed instance in your cluster.
- 🔄 **helm upgrade / rollback** = Safely manage the entire app lifecycle.
- 🗑️ **helm uninstall** = Deletes all associated resources instantly.

> Next → [Metrics Server & Monitoring](06-metrics-server-monitoring.md) 🚀
