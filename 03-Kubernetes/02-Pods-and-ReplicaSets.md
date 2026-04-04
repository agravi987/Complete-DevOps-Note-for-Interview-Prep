# Pods and ReplicaSets

## 1. Overview
In Docker, the smallest unit of deployment is a Container. In Kubernetes, the smallest unit of deployment is a **Pod**. A Pod is a wrapper around one or more tightly coupled containers. While Pods are the core execution layer, a **ReplicaSet** is the supervisor that ensures a specific number of exact Pod clones are running at all times to guarantee application availability.

## 2. Why This Matters
- **Shared Context:** Sometimes containers *must* live together. For example, a web server container and a log-scraping container. Placing them in the same Pod allows them to share the exact same Localhost IP and Volume storage natively.
- **Self-Healing Infrastructure:** If a node crashes and takes down 3 Pods, the ReplicaSet immediately detects that the actual count dropped below the desired count, and instantly boots 3 new Pods on healthy nodes to compensate.
- **Horizontal Scaling:** When traffic spikes, you simply tell the ReplicaSet to increase its count from 5 to 50.

## 3. Core Concepts
- **Pod:** The atomic, disposable execution unit. It gets a unique internal IP address.
- **Multi-Container Pod (Sidecar Pattern):** Running a primary app container and a secondary helper container (like a metrics exporter) cleanly packaged inside a single Pod.
- **ReplicaSet (RS):** A specialized controller whose sole purpose is to maintain a stable set of identically configured Pods.
- **Labels & Selectors:** The "glue" K8s uses. A Pod has a label (`app: frontend`). A ReplicaSet has a selector (`matchLabels: app: frontend`). If it finds 3 pods with that label, it manages them.

## 4. Architecture / Workflow
1. **Definition:** You submit a YAML manifest for a ReplicaSet requesting `replicas: 3` with the label `tier: web`.
2. **Current State Check:** The ReplicaSet controller surveys the cluster for any existing Pods with the label `tier: web`. It finds `0`.
3. **Creation:** The RS instructs the API Server to create 3 brand new Pods using the template provided in the YAML.
4. **Maintenance Loop:** An hour later, an engineer manually deletes one of the Pods. The RS loop immediately notices the count is now `2`, which does not match the desired `3`. It instantly creates a replacement Pod.

## 5. Commands & Practical Usage

Find all Pods, including their internal IP addresses and which Node they are sitting on:
```bash
kubectl get pods -o wide
```

See the reason *why* a Pod is failing to start:
```bash
kubectl describe pod <pod-name>
# Scroll to the bottom to see the "Events" section.
```

Check the logs of an application inside a Pod:
```bash
kubectl logs <pod-name>
# If there are multiple containers in the pod, you must specify which one:
# kubectl logs <pod-name> -c <container-name>
```

Get all ReplicaSets to see if desired matches current:
```bash
kubectl get rs
```

Manually scale a ReplicaSet temporarily:
```bash
kubectl scale rs my-app-replicaset --replicas=5
```

## 6. Configuration / YAML / Code Examples

A classic ReplicaSet configuration ensuring 3 instances of Nginx are always running:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-rs
spec:
  # 1. How many clones do we want?
  replicas: 3
  # 2. How does the RS know which pods belong to it?
  selector:
    matchLabels:
      app: web-frontend
  # 3. If a new pod is needed, what should it look like? (The blueprint)
  template:
    metadata:
      # This label MUST match the selector above!
      labels:
        app: web-frontend
    spec:
      containers:
      - name: nginx-server
        image: nginx:1.21.0
        ports:
        - containerPort: 80
```

## 7. Hands-on Step-by-Step (VERY IMPORTANT)

**Step 1: Apply the ReplicaSet YAML**
Save the code above as `rs.yaml` and execute:
```bash
kubectl apply -f rs.yaml
```

**Step 2: Verify the Pods spawned**
```bash
kubectl get pods
```
> *You will see 3 pods named something like `frontend-rs-a5x89`, `frontend-rs-b9z21`.*

**Step 3: Test the Self-Healing mechanism**
Delete one of the pods manually (simulating a crash):
```bash
kubectl delete pod frontend-rs-a5x89
```

**Step 4: Watch the resurrection**
Immediately run `kubectl get pods` again. You'll see the old one terminating, and a brand new one already transitioning to `ContainerCreating`.

**Step 5: Isolate a bad Pod (Pro-tip)**
Sometimes a pod has a bug. If you delete it, K8s restarts it, destroying the evidence. Instead, just *edit* the pod and change its label:
```bash
kubectl label pod frontend-rs-b9z21 app=isolated --overwrite
```
> *Because its label no longer matches `app=web-frontend`, the ReplicaSet abandons it (but leaves it running for you to debug!). The RS then instantly creates a new one to maintain the count of 3.*

## 8. Common Errors & Troubleshooting

- **Error: Pod stuck in `Pending` state indefinitely**
  - **Issue:** The K8s Scheduler cannot find a worker node with enough free CPU/RAM to host the pod, or the pod requested a feature that no node possesses (e.g. GPU).
  - **Fix:** Use `kubectl describe pod <name>` to read the scheduler events. Add a new node to the cluster or reduce the Pod's resource requests.
- **Error: ReplicaSet keeps creating pods, but they all go into `CrashLoopBackOff`**
  - **Issue:** The application code inside the container is throwing a fatal error and exiting. The RS keeps restarting it helplessly.
  - **Fix:** Read the application logs via `kubectl logs <pod-name>`. Fix the code and push a new image.
- **Error: You update the `image` in the ReplicaSet YAML, but K8s does nothing when you apply it.**
  - **Issue:** ReplicaSets DO NOT support rolling infrastructure updates natively. They will only use the new image template for *future* pod replacements, they won't update existing pods.
  - **Fix:** This is exactly why we use **Deployments** instead of raw ReplicaSets!

## 9. Best Practices

1. **Never deploy raw ReplicaSets to Production:** Always wrap a ReplicaSet inside a **Deployment**. Deployments manage ReplicaSets directly and provide version control, rollbacks, and zero-downtime rolling updates.
2. **One App per Pod Rule:** Unless containers are physically required to share a filesystem (Sidecar pattern), do not put a Web Server and a Database in the same Pod. They should scale independently.
3. **Use Liveness Probes:** A pod can have a status of `Running` even if the Java app inside is completely deadlocked. Add a Liveness HTTP Probe to the Pod spec so K8s knows exactly when to execute `kill()`.

## 10. Interview Questions & Answers

**Q1: Why does K8s use Pods instead of just managing Docker Containers directly?**
**A1:** Pods act as an abstraction layer. It allows K8s to decouple itself tightly from Docker. If you switch your container runtime from Docker to CRI-O, K8s doesn't care because it only manages the "Pod" abstraction. It also natively solves the "shared network/shared volume" requirement for tightly coupled processes (sidecars).

**Q2: What happens if you manually delete a Pod managed by a ReplicaSet?**
**A2:** The ReplicaSet continuously monitors the label selectors. It instantly observes that the count has dropped below the desired `replicas` integer and immediately requests the API Server to spin up a replacement Pod.

**Q3: Two containers running inside the exact same Pod need to communicate. How do they do it?**
**A3:** Because containers in a single Pod share the exact same network namespace, they can communicate purely by calling `localhost:<port>`. 

**Q4: Can a ReplicaSet span across multiple worker nodes?**
**A4:** The ReplicaSet controller itself runs on the Control Plane. The actual Pods it creates are deliberately scattered across multiple worker nodes by the Scheduler to guarantee High Availability.

**Q5: What is the main limitation of a ReplicaSet compared to a Deployment?**
**A5:** ReplicaSets cannot natively perform zero-downtime rolling updates or rollbacks. If you change a ReplicaSet's YAML image version, it will not restart the existing pods to apply the new image. 

## 11. Quick Revision Summary
- **Pod:** The smallest deployable unit. Shares localhost IP & Storage.
- **ReplicaSet:** The silent guardian prioritizing high-availability. Ensures X pods match Y labels continuously.
- **Labels/Selectors:** The core mechanism connecting the ReplicaSet to its target Pods.
- **Sidecar:** A helper container running alongside the main app in the exact same Pod.

## 12. Official Documentation Links
- [Kubernetes Pods Concept](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Kubernetes ReplicaSet Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
