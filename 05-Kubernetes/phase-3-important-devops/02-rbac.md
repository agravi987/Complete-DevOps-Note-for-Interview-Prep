# 🔐 Role-Based Access Control (RBAC)

> **Hey Ravi! 👋** You wouldn't give the new intern the keys to the production database, right? In Kubernetes, if you don't set up **RBAC**, everyone has the keys to everything! 😱 RBAC lets you say exactly WHO can do WHAT to WHICH resources. Let's lock this cluster down! 🔒

---

## 🧠 What is RBAC?

> **Role-Based Access Control determines whether a specific user (or program) is allowed to perform a specific action (like `delete`) on a specific resource (like `pods`) in a specific namespace.**

By default in Kubernetes, all permissions are **DENIED**. You have to explicitly grant them.

---

## 🪪 The VIP Club Analogy

1. **User/ServiceAccount (The Person):** "Hi, I'm Ravi."
2. **Role (The VIP Badge):** A badge that says "Can enter the VIP Lounge and drink free soda."
3. **RoleBinding (The Bouncer's List):** The list the bouncer holds that says "Ravi is allowed to wear the VIP Badge." 📋

*You need BOTH the Role (what you can do) and the RoleBinding (who gets to do it) for it to work!*

---

## 🎭 The 4 Objects of RBAC

| Object | Scope | What it does |
|---|---|---|
| **Role** | 1 Namespace | Defines permissions (e.g., "Can get pods in 'dev' namespace"). |
| **RoleBinding** | 1 Namespace | Attaches a `Role` to a User/Group/ServiceAccount in that namespace. |
| **ClusterRole** | Entire Cluster | Defines permissions globally (e.g., "Can list nodes across all namespaces"). |
| **ClusterRoleBinding**| Entire Cluster | Attaches a `ClusterRole` to someone globally. |

> 💡 **Ravi's Rule:** If you want to give someone permission in ONE namespace, use Role + RoleBinding. If you want to give them permission EVERYWHERE, use ClusterRole + ClusterRoleBinding.

---

## 📄 Real YAML (The "Read-Only" Developer)

Let's say we want to let Ravi VIEW pods in the `dev` namespace, but not delete them!

### 1. Create the Role (The Permissions)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]           # "" means the core API group
  resources: ["pods", "logs"]
  verbs: ["get", "watch", "list"]   # NO 'create' or 'delete'!
```

### 2. Create the RoleBinding (Attach it to Ravi)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-ravi
  namespace: dev
subjects:
- kind: User                # It could also be a Group or ServiceAccount
  name: ravi
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 🎮 Commands & Testing (Auth Can-I)

You can literally ask Kubernetes: "Am I allowed to do this?"

```bash
# Can I delete pods in the dev namespace?
kubectl auth can-i delete pods -n dev
# Output: yes (or no)

# As an admin, check if Ravi can delete pods:
kubectl auth can-i delete pods -n dev --as=ravi

# List roles and bindings
kubectl get roles,rolebindings -n dev
kubectl get clusterroles
```

---

## ⚠️ Common API Groups & Verbs

You must know these for the YAML!

**Verbs (The Actions):**
- `get` (Read one specific item)
- `list` (Read all items)
- `watch` (Stream live updates)
- `create`, `update`, `patch`, `delete` (Write actions)
- `*` (Wildcard: ALL actions!)

**API Groups (The Folders):**
- `""` (Core group: Pods, Services, ConfigMaps, Secrets)
- `"apps"` (Deployments, StatefulSets, ReplicaSets)
- `"networking.k8s.io"` (Ingress, NetworkPolicies)
- `"batch"` (Jobs, CronJobs)

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between a Role and a ClusterRole?**
> "A Role is namespace-scoped, meaning its permissions only apply within a specific namespace. A ClusterRole is cluster-scoped, meaning it can grant permissions across all namespaces (like viewing Deployments everywhere) or grant permissions to cluster-level resources that don't belong to a namespace (like Nodes or PersistentVolumes)."

**Q: How do you grant a user permission to list Pods in a specific namespace?**
> "You need to create two resources: First, a `Role` in that namespace with a rule that specifies `apiGroups: [""]`, `resources: ["pods"]`, and `verbs: ["list"]`. Second, a `RoleBinding` in that same namespace that binds that `Role` to the specific `User` subject."

**Q: Why does the `roleRef` in a RoleBinding exist? Why not just put the permissions directly in the RoleBinding?**
> "Separation of concerns and reusability. By keeping the permissions in a `Role`, you can define 'View-Only' once, and then use multiple `RoleBindings` to assign that exact same 'View-Only' role to different users or groups independently without duplicating the permission logic."

---

## ⚡ 30-Second Revision

- 🔒 **RBAC** = Default Deny! You must explicitly grant access.
- 📜 **Role** = What you can do (in ONE namespace).
- 🖇️ **RoleBinding** = Who gets to do it (in ONE namespace).
- 🌍 **ClusterRole** = What you can do (Cluster-wide, or for Nodes).
- 🖇️ **ClusterRoleBinding** = Who gets to do it (Cluster-wide).
- 🔑 **Verbs:** get, list, watch, create, delete, update.
- ❓ `kubectl auth can-i` is your best debugging tool!

> Next → [Service Accounts](03-service-accounts.md) 🚀
