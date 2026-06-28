# 🏗️ Custom Resource Definitions (CRDs)

> **Hey Ravi! 👋** Kubernetes comes with built-in objects: Pods, Deployments, Services. But what if you want Kubernetes to manage your *own* custom things? What if you want to write a YAML file that says `kind: Database` or `kind: Pizza`? 🍕 You can! You just have to teach Kubernetes a new word. That's what **CRDs** do! 📚

---

## 🧠 What is a CRD?

> **A Custom Resource Definition (CRD) extends the Kubernetes API. It allows you to create entirely new resource `kinds` that act exactly like native Kubernetes objects.**

- **CRD (The Dictionary Definition):** Teaches K8s what a "Pizza" is and what toppings are allowed.
- **CR (Custom Resource - The Actual Object):** An actual YAML file you apply saying "Make me a Pepperoni Pizza."

---

## 📖 The Dictionary Analogy

Imagine Kubernetes is a person who only speaks basic English (Pods, Services).
1. You hand them a **CRD** (A dictionary page). It says: *"A 'DatabaseCluster' is a new word. It must have a 'storageSize' (number) and an 'engine' (string)."*
2. Now K8s knows the word!
3. Later, a developer hands K8s a **CR** (A YAML file saying `kind: DatabaseCluster`). 
4. K8s reads it, checks the dictionary to make sure the spelling and types are correct, and accepts it! ✅

---

## 📄 Real YAML (Creating a CRD)

Let's teach Kubernetes a new object called `MySQLCluster`.

### 1. The CRD (Admin creates this once)
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: mysqlclusters.database.ravi.com
spec:
  group: database.ravi.com
  names:
    kind: MySQLCluster      # The new 'kind' we are creating!
    plural: mysqlclusters   # How to query it (kubectl get mysqlclusters)
    singular: mysqlcluster
    shortNames:
    - msql                  # kubectl get msql
  scope: Namespaced         # Is it per-namespace or cluster-wide?
  versions:
  - name: v1
    served: true
    storage: true
    schema:                 # VALIDATION! What fields are allowed?
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              replicas:
                type: integer   # Must be a number!
                minimum: 1
              storageGB:
                type: integer
```

### 2. The Custom Resource (Developer uses this daily)
```yaml
apiVersion: database.ravi.com/v1
kind: MySQLCluster          # 🪄 LOOK! We are using our new kind!
metadata:
  name: prod-db
spec:
  replicas: 3               # This will be validated by the CRD!
  storageGB: 100
```

---

## ⚠️ The Big Catch: CRDs Don't DO Anything!

If you apply the `MySQLCluster` YAML above, Kubernetes will accept it, save it to the `etcd` database, and... **nothing will happen.** 😳

Why? Because a CRD is just *data storage*. Kubernetes doesn't actually know HOW to build a MySQL Cluster! 
To make the CRD actually DO something (like spin up pods and volumes based on that YAML), you need to write a custom program that watches for that CRD. 

That program is called an **Operator** (or Custom Controller). *(We cover this in the next topic!)*

---

## 🎮 Commands

```bash
# Get all CRDs installed in the cluster
kubectl get crds

# Find our new one
kubectl get crd mysqlclusters.database.ravi.com

# Create an actual instance of our new object!
kubectl apply -f my-database.yaml

# Query it using the shortName we defined!
kubectl get msql
# Output:
# NAME      AGE
# prod-db   2m
```

---

## 🎤 Interview Knockout Answers

**Q: What is a Custom Resource Definition (CRD)?**
> "A CRD is a powerful feature that allows you to extend the Kubernetes API by defining your own custom resource types. Once a CRD is registered, users can create and manage Custom Resources (CRs) using standard `kubectl` commands, just like they would with native objects like Pods or Deployments."

**Q: What is OpenAPI validation in a CRD?**
> "When defining a CRD, you provide an OpenAPI v3 schema. This acts as strict validation for any Custom Resources created by users. The Kubernetes API server will automatically reject any YAML that doesn't match the schema, for example, if a user provides a string where an integer is required for 'replicas'."

**Q: If I create a CRD and then create a Custom Resource from it, what actually happens in the cluster?**
> "The Kubernetes API server validates the Custom Resource against the CRD's schema and stores the object in the `etcd` database. However, no actual action is taken (no pods are created) unless you have a Custom Controller or Operator running in the cluster that is specifically programmed to watch for that Custom Resource and act upon it."

---

## ⚡ 30-Second Revision

- 📚 **CRD** = Teaches K8s a new API object (`kind: MySQLCluster`).
- 📝 **CR** = The actual YAML instance you create using that new kind.
- 🛡️ CRDs enforce strict **Validation** (OpenAPI schema) to ensure users don't put bad data in the YAML.
- 🗄️ CRDs only store data in the K8s database (`etcd`).
- 🤖 To make a CRD actually perform actions, you MUST pair it with an **Operator**.

> Next → [Operators](03-operators.md) 🚀
