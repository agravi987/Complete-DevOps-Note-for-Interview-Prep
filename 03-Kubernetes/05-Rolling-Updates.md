# Kubernetes Rolling Updates

## 1. Overview
A **Rolling Update** is the default deployment mechanism in Kubernetes used to update the running version of your application (like bumping a Docker image tag) with zero downtime. Instead of stopping all instances of your app simultaneously and then bringing up the new ones, Kubernetes incrementally replaces the old Pods with the new Pods one by one.

## 2. Why This Matters
- **Zero Downtime Deployments:** Your users won't experience service interruptions while the application is updating in the background.
- **Controlled Rollouts:** You can control the exact mathematical pace of the update (how many pods are taken down and created at the same time).
- **Risk Mitigation:** If the newly deployed version crashes, the rollout pauses, preventing the broken version from receiving a majority of your user traffic.
- **Instant Rollbacks:** If something goes fundamentally wrong, Kubernetes allows you to immediately revert to the previous working state via historical ReplicaSets.

## 3. Core Concepts
- **Deployment:** The high-level Kubernetes object that manages the underlying ReplicaSets and provides declarative updates to Pods.
- **ReplicaSet:** Ensures that a specified number of pod replicas are actively running. During an update, a brand new ReplicaSet is spun up alongside the old one.
- **`maxSurge`:** The maximum number of Pods that can be created *over* the desired number of Pods during an update (e.g., if desired=10, and `maxSurge=25%`, K8s can scale up to 13 pods during the rollout).
- **`maxUnavailable`:** The maximum number of Pods that can be *unavailable* during the update process (e.g., if desired=10 and `maxUnavailable=25%`, K8s guarantees at least 8 pods remain active).

## 4. Architecture / Workflow
1. **Trigger:** The engineer updates the Deployment configuration (usually changing the container image version).
2. **New ReplicaSet Created:** The Deployment creates a *new* empty ReplicaSet for the new version.
3. **Scale Up New:** The new ReplicaSet is scaled up gradually based on the `maxSurge` definition.
4. **Scale Down Old:** The *old* ReplicaSet is slowly scaled down based on the `maxUnavailable` definition.
5. **Repeat Loop:** This process loops until all new Pods are running healthily, and the old ReplicaSet is scaled entirely down to `0`.

## 5. Commands & Practical Usage

Update the image of a deployment directly from the CLI:
```bash
kubectl set image deployment/my-app nginx=nginx:1.16.1 --record
```
> *Pro-Tip: The `--record` flag saves the command executed in the rollout history, making tracking easier.*

Check the real-time rollout status:
```bash
kubectl rollout status deployment/my-app
```
> *This blocks your terminal and watches the rollout happen step-by-step.*

View rollout history to find old revisions:
```bash
kubectl rollout history deployment/my-app
```

Undo a rollout (Instant Rollback):
```bash
kubectl rollout undo deployment/my-app
```
> *Immediately rolls back to the previous revision if the new version is failing.*

## 6. Configuration / YAML / Code Examples

A robust, production-grade Deployment YAML defining a customized rolling update strategy:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:
    app: frontend
spec:
  replicas: 4
  selector:
    matchLabels:
      app: frontend
  # The Strategy Block defines the Rollout Behavior
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%        # Can exceed desired by 1 Pod (25% of 4)
      maxUnavailable: 25%  # At least 3 Pods guarantee to be running
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        # Readiness Probe is CRITICAL for zero-downtime
        readinessProbe:    
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 7. Hands-on Step-by-Step (VERY IMPORTANT)

**Step 1: Deploy Initial Version**
Save the YAML from Section 6 as `deployment.yaml` and apply it:
```bash
kubectl apply -f deployment.yaml
```
Verify the 4 pods are running successfully:
```bash
kubectl get pods
```

**Step 2: Trigger the Rolling Update**
Change the image version from `1.14.2` to `1.16.1` and explicitly record the change:
```bash
kubectl set image deployment/frontend-deployment frontend=nginx:1.16.1 --record
```

**Step 3: Monitor the Rollout**
Watch K8s orchestrate the replacement in real-time:
```bash
kubectl rollout status deployment/frontend-deployment
```

**Step 4: Verify ReplicaSets post-rollout**
Notice that the old ReplicaSet is left intact but scaled to `0`. It acts as a backup!
```bash
kubectl get rs
```

**Step 5: Induce a Failure & Perform a Rollback**
Let's intentionally update to an image tag that doesn't exist to simulate a broken release:
```bash
kubectl set image deployment/frontend-deployment frontend=nginx:does-not-exist
```
Notice the rollout gets stuck because Kubernetes is protecting your infrastructure from taking down the healthy pods:
```bash
kubectl rollout status deployment/frontend-deployment
```
Rollback to the last working version instantly:
```bash
kubectl rollout undo deployment/frontend-deployment
```

## 8. Common Errors & Troubleshooting

- **Error: `ImagePullBackOff` or `ErrImagePull` during rollout**
  - **Issue:** The new version's docker image is misspelled, not uploaded, or missing registry authentication.
  - **Fix:** The rollout will safely pause. K8s will keep the old pods running! Fix the image tag via `kubectl edit deployment/frontend-deployment` or rollback using the undo command.
  
- **Error: New pods are instantly crashing (`CrashLoopBackOff`)**
  - **Issue:** Your new application code is throwing a fatal error on boot. 
  - **Fix:** K8s will halt the rollout. Check logs using `kubectl logs <new-pod-name>`. Address the application bug, or `kubectl rollout undo`. 
  
- **Error: Rollout completes but old connections break unexpectedly**
  - **Issue:** Your application requires a graceful shutdown but your containers are being forcefully terminated instantly.
  - **Fix:** Implement `terminationGracePeriodSeconds` in your pod spec and ensure you handle `SIGTERM` signals properly in your application's entrypoint code.

## 9. Best Practices

1. **Always use Custom Readiness Probes:** If K8s doesn't know when your application is *actually* ready to receive traffic (e.g. database connections are established), it might route traffic to a pod that's booted up but hasn't loaded its configuration yet, resulting in 502/504 errors.
2. **Never deploy the `latest` tag:** Always use specific semantic versions or Git SHAs (like `v1.2.3` or `f7a91b`). If you use `:latest`, K8s might not detect an image change when you re-apply, and it makes historical rollbacks completely unpredictable.
3. **Configure Resource Requests/Limits:** If your new pods try to schedule but there isn't enough CPU/RAM on the worker nodes, the rollout will hang in a `Pending` state.

## 10. Interview Questions & Answers

**Q1: How does Kubernetes guarantee "zero downtime" during a Rolling Update?**
**A1:** It maintains dual ReplicaSets. It spins up new pods while ensuring a minimum number of old pods (`maxUnavailable`) are still serving traffic. New traffic is only routed to the new pods once they successfully pass their configured Readiness Probes.

**Q2: What is the mechanical difference between `maxSurge` and `maxUnavailable`?**
**A2:** `maxSurge` controls how many *extra* pods K8s is permitted to create above the desired replica count during the update. `maxUnavailable` dictates how many pods can be taken down below the desired replica count during the update.

**Q3: If a Rolling Update introduces a critical application bug causing pods to crash, will production go fully offline?**
**A3:** No. Kubernetes actively evaluates the health of the newly rolling pods. If they crash (`CrashLoopBackOff`), the rolling update will permanently pause, and K8s refuses to take down the remaining functionally healthy older pods.

**Q4: How do you immediately stop a bad rollout and revert?**
**A4:** Execute `kubectl rollout undo deployment/<deployment-name>`. The Deployment controller will immediately switch back to the previous ReplicaSet revision.

**Q5: Why is the Readiness Probe critical for the success of Rolling Updates?**
**A5:** Without a custom Readiness Probe, Kubernetes blindly assumes a pod is fully ready to accept user traffic the millisecond the container process starts. This causes the Service to route live traffic to apps that are still downloading config or making DB connections, leading to dropped end-user requests.

## 11. Quick Revision Summary
- **Rolling Update:** Default K8s strategy, facilitating zero-downtime updates.
- **Mechanism:** Safely scales down older ReplicaSet while deliberately scaling up a newly generated one.
- **Key Tuning Settings:** `maxSurge` (going over capacity) and `maxUnavailable` (going under capacity).
- **Golden Rule:** ALWAYS pair rollouts with Readiness Probes to functionally protect traffic flow.
- **Emergency Button:** `kubectl rollout undo`.

## 12. Official Documentation Links
- [Kubernetes Deployments Concept Guide](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Performing a Rolling Update Tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
