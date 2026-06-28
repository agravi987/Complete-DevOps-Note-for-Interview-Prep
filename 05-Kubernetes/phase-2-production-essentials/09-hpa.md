# 📈 Horizontal Pod Autoscaler (HPA)

> **Hey Ravi! 👋** Black Friday is here. Traffic to your store spikes 1000%. If you only have 3 pods running, your app crashes and you lose money. 📉 But what if K8s could watch the CPU usage, and automatically scale your Deployments up to 50 pods when traffic spikes, and back down to 3 when it's quiet? That's the magic of **HPA**! 🪄

---

## 🧠 What is HPA?

> **Horizontal Pod Autoscaler automatically updates a workload resource (like a Deployment or StatefulSet), aiming to automatically scale the number of Pods to match demand.**

- **Horizontal Scaling:** Adding MORE pods. (HPA does this).
- **Vertical Scaling:** Giving existing pods MORE CPU/RAM. (VPA does this - less common).

---

## 🌡️ The Thermostat Analogy

- You set your AC thermostat to 72°F (Target CPU = 50%).
- The room gets hot because 10 people walk in (Traffic spikes, CPU hits 90%).
- The AC kicks into high gear (HPA adds 5 more pods).
- The room cools down to 72°F (Load balances across 8 pods, CPU drops to 50%).
- People leave, room gets freezing cold (CPU hits 10%).
- AC turns down (HPA deletes extra pods). ❄️

---

## ⚠️ The Golden Requirement: Metrics Server

HPA is blind by default! It doesn't know how much CPU your pods are using. 
You **MUST** install a tool called **Metrics Server** in your cluster for HPA to work.

Also, your Deployment **MUST** have resource `requests` defined. (If a pod doesn't request a specific amount of CPU, K8s can't calculate a "percentage" of usage!)

---

## 📄 Real YAML (HPA v2)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app           # The deployment we want to scale!
  
  minReplicas: 2            # Never drop below 2 pods
  maxReplicas: 10           # Never go above 10 pods (protect your cloud bill!)
  
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60   # Target 60% CPU usage across all pods
```

---

## ⏱️ Stabilization Windows (Cooldowns)

If traffic spikes for 2 seconds, we don't want to spin up 50 pods instantly. If traffic drops for 2 seconds, we don't want to kill them. This "flapping" is bad.

HPA has built-in cooldowns:
- **Scale Up:** Quick (usually responds within seconds/minutes).
- **Scale Down:** Slow (usually waits 5 minutes to ensure the traffic drop is real).

---

## 🎮 Commands & Testing

```bash
# Check your HPA status
kubectl get hpa
# Look at the TARGETS column: 
# 0%/60%  (Current usage is 0, target is 60)
# <unknown>/60%  (Uh oh! Metrics server is broken or Pods lack resource requests!)

# See detailed scaling events
kubectl describe hpa web-app-hpa

# 🔥 FUN TEST: Create a load generator to test your HPA!
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never \
  -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://web-app-svc; done"

# Open another terminal and watch it scale!
kubectl get hpa -w
```

---

## 🎤 Interview Knockout Answers

**Q: How does the Horizontal Pod Autoscaler (HPA) work?**
> "HPA continuously monitors metrics (like CPU or Memory) via the Metrics Server API. It compares the current average utilization against a target threshold. If utilization exceeds the target, HPA modifies the `replicas` count on the target Deployment or StatefulSet to scale out. When load decreases, it scales in."

**Q: I created an HPA but the target shows `<unknown>`. Why?**
> "There are two main reasons for this. First, the cluster might not have the Metrics Server installed. Second, the target Deployment's Pods might not have CPU/Memory `requests` defined in their YAML. HPA needs a baseline request to calculate percentage utilization."

**Q: Can HPA scale based on things other than CPU and Memory?**
> "Yes, using HPA v2 and Custom Metrics. You can scale based on application-specific metrics like 'HTTP requests per second' or 'queue length in RabbitMQ' by using Prometheus and an external metrics adapter."

---

## ⚡ 30-Second Revision

- 📈 **HPA** = Auto-scales pod count based on load.
- 📉 Requires **minReplicas** and **maxReplicas** (guardrails).
- ⚠️ **Metrics Server** MUST be installed in the cluster.
- 📦 Pods MUST have `resources.requests` defined in their YAML.
- ⏱️ **Cooldowns** prevent "flapping" (scaling up and down too rapidly).
- 🔍 `kubectl get hpa` → Watch the `TARGETS` column to debug.

---

> Next → [Troubleshooting Guide](10-interview-troubleshooting.md) 🚀
