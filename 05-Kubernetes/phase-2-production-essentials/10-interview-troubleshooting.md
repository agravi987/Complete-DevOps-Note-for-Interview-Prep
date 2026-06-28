# 🩺 BONUS: The Ultimate Troubleshooting Guide

> **Hey Ravi! 👋** You've learned all the concepts, but in a real Senior DevOps interview, they rarely ask "What is a Pod?" Instead, they ask: *"A developer deployed an app, and it's stuck in CrashLoopBackOff. Walk me through exactly how you fix it."* 
> 
> This bonus guide contains the real-world debugging workflows you MUST know by heart! 🕵️‍♂️🔍

---

## 🚨 Scenario 1: `Pending` Pods

**The Symptom:** `kubectl get pods` shows a pod stuck in `Pending` for minutes.

**The Workflow:**
1. **Command:** `kubectl describe pod <pod-name>` 
2. **Look at:** The `Events` section at the very bottom.
3. **Common Culprits:**
   - **Insufficient Resources:** `"0/3 nodes are available: 3 Insufficient cpu."` (Fix: Add more nodes via Cluster Autoscaler, or lower the CPU `requests` in the YAML).
   - **PVC Issues:** `"pod has unbound immediate PersistentVolumeClaims."` (Fix: Check if the StorageClass exists or if the PV is available using `kubectl get pvc`).
   - **Taints/Affinities:** The pod wants to land on a node with `gpu=true`, but no such node exists.

---

## 💥 Scenario 2: `CrashLoopBackOff` or `Error`

**The Symptom:** The pod starts, crashes, and Kubernetes keeps restarting it with increasing delays.

**The Workflow:**
1. **Command 1:** `kubectl logs <pod-name>`
   - *Pro-Tip:* If the pod restarts too fast, check the previous crashed instance: `kubectl logs <pod-name> --previous`
   - Look for application errors (e.g., "Cannot connect to database", "Missing environment variable").
2. **Command 2:** `kubectl describe pod <pod-name>`
   - Look at the `State` and `Reason`.
3. **Common Culprits:**
   - **OOMKilled:** The container used more RAM than its `limits`. (Fix: Increase memory limit or fix memory leak).
   - **Liveness Probe Failed:** The health check failed, so the Kubelet murdered the pod. (Fix: Increase `initialDelaySeconds` or fix the app).
   - **Bad Entrypoint:** A typo in the Docker `CMD` or Kubernetes `command`. (Error usually says "executable file not found in $PATH").

---

## 👻 Scenario 3: `ImagePullBackOff` or `ErrImagePull`

**The Symptom:** The pod can't even start because it can't download the Docker image.

**The Workflow:**
1. **Command:** `kubectl describe pod <pod-name>`
2. **Common Culprits:**
   - **Typo in Image Name:** You typed `ngnx` instead of `nginx`.
   - **Typo in Image Tag:** The tag `v1.0.5` doesn't exist in the registry.
   - **Authentication Issue:** You are pulling from a private registry (like AWS ECR) but forgot to provide an `imagePullSecrets` in the YAML!

---

## 🚧 Scenario 4: Pod is `Running` but unreachable

**The Symptom:** The Pod is perfectly healthy, but the web browser just spins, or `curl` returns a timeout.

**The Workflow:**
1. **Check the Readiness Probe:** `kubectl describe pod <name>`
   - If the Readiness probe is failing, the Pod is removed from the Service endpoints. (Fix the app or the probe path).
2. **Check the Service Endpoints:** `kubectl get endpoints <service-name>`
   - Does it list IP addresses? If it says `<none>`, your Service `selector` labels DO NOT MATCH the Pod's labels! (Classic mistake!) 🤦‍♂️
3. **Check the Port:** Are you targeting `targetPort: 8080` in the Service, but the app inside the container is actually listening on `80`? 
4. **Check Network Policies:** Is there a `NetworkPolicy` blocking incoming traffic?

---

## 🎤 The "STAR" Interview Question Framework

When the interviewer asks: *"Tell me about a time you fixed a complex Kubernetes outage."* 
Use the **STAR** method to sound like a Senior Engineer:

- 🌟 **Situation:** "In production, our main checkout API suddenly started dropping 50% of traffic."
- 🎯 **Task:** "I needed to identify the root cause immediately without causing further downtime."
- 🏃‍♂️ **Action:** "I ran `kubectl get pods` and noticed half the pods were restarting. I ran `kubectl describe` and saw `OOMKilled`. I realized a recent code push introduced a memory leak. I used `kubectl rollout undo` to instantly revert to the previous stable version. Then, I adjusted the HPA to handle the load while developers fixed the leak."
- 🏆 **Result:** "Downtime was resolved in 2 minutes, and we implemented stricter memory profiling in our CI pipeline to prevent it from happening again."

---

## ⚡ Your Golden Command Cheat Sheet

```bash
# 1. Why won't it start?
kubectl describe pod <name>

# 2. Why did it crash?
kubectl logs <name> --previous

# 3. Why isn't traffic reaching it?
kubectl get endpoints <service-name>

# 4. Jump inside and test the network directly!
kubectl exec -it <pod-name> -- curl http://database-svc:3306

# 5. Who is eating all the RAM?
kubectl top pods --sort-by=memory
```

> **You are ready, Ravi! Go crush that interview!** 🚀

---

> **Woohoo! Phase 2 is COMPLETE! 🎉** You just leveled up your production skills massively. Ready for the deep DevOps stuff? 
> **Back to:** [Phase 2 README](README.md) | **Next Phase:** [Phase 3 - Important for DevOps](../phase-3-important-devops/README.md) 🚀
