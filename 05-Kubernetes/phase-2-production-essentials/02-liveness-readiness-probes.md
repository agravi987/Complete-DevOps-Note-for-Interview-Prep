# 🩺 Liveness, Readiness, & Startup Probes

> **Hey Ravi! 👋** Have you ever had an app that was technically "running" but completely frozen? Or an app that just started, but wasn't ready to serve traffic yet? Kubernetes can't read your app's mind. You have to TEACH it how to check your app's health. That's what Probes do! 👩‍⚕️

---

## 🧠 What are Probes?

> **Probes are health checks. Kubernetes pings your app periodically to ask "Are you alive?" or "Are you ready?" and takes action based on the answer.**

---

## 🚑 The Paramedic, The Bouncer, and The Alarm Clock

### 1️⃣ Liveness Probe (The Paramedic) 🩺
- **Question:** "Are you dead or frozen?"
- **Action if failing:** Kills the pod and restarts it! 💀🔁
- **Use case:** Your app is stuck in a deadlock and needs a hard reboot.

### 2️⃣ Readiness Probe (The Bouncer) 🛑
- **Question:** "Are you ready to handle traffic right now?"
- **Action if failing:** Removes the pod from the Service endpoints. (No traffic sent to it!) 🚧
- **Use case:** Your app is loading a massive database into memory and can't serve users yet. (Or it's too busy right now!)

### 3️⃣ Startup Probe (The Alarm Clock) ⏰
- **Question:** "Have you finished your slow startup sequence?"
- **Action if failing:** Kills the pod.
- **Bonus:** Disables Liveness/Readiness probes until it succeeds!
- **Use case:** Legacy Java apps that take 3 minutes to start. (Prevents Liveness probe from killing it prematurely).

---

## 🏗️ How Probes Work (3 Types)

You can check health in 3 ways:

1. **HTTP GET:** `curl http://pod:8080/healthz` (200-399 = Success!)
2. **TCP Socket:** Can I open a TCP connection on port 3306? (Great for databases!)
3. **Exec Command:** Run `cat /tmp/healthy` inside the container. (Exit code 0 = Success!)

---

## 📄 Full Probe YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
  - name: my-app
    image: my-slow-java-app:v1
    ports:
    - containerPort: 8080

    # ⏰ 1. STARTUP PROBE: Wait up to 3 mins for app to start
    startupProbe:
      httpGet:
        path: /health/startup
        port: 8080
      failureThreshold: 30     # 30 * 10s = 300 seconds max!
      periodSeconds: 10

    # 🛑 2. READINESS PROBE: Check if ready to receive traffic
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 8080
      initialDelaySeconds: 5   # Wait 5s before first check
      periodSeconds: 5         # Check every 5s
      successThreshold: 1      # 1 success = ready
      failureThreshold: 3      # 3 failures = unready

    # 🩺 3. LIVENESS PROBE: Check if frozen
    livenessProbe:
      tcpSocket:               # Let's use TCP this time!
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 10
```

### 🎛️ The Knobs and Dials
- `initialDelaySeconds`: Wait this long before starting checks.
- `periodSeconds`: How often to check.
- `timeoutSeconds`: How long to wait for a reply before giving up.
- `successThreshold`: Consecutive successes needed to be "healthy".
- `failureThreshold`: Consecutive failures needed to take action.

---

## 🎮 Troubleshooting Commands

```bash
# See probe failures in events!
kubectl describe pod probe-demo

# Typical event output:
# Warning  Unhealthy  Liveness probe failed: HTTP probe failed with statuscode: 500
# Warning  Unhealthy  Readiness probe failed: connection refused

# Check if Readiness probe is failing:
kubectl get endpoints my-service
# If the IPs are missing, the Readiness probe is failing!
```

---

## 🚨 Common Gotchas & Mistakes

- ❌ **Using Liveness without Readiness:** Your app might restart forever if it just needed a moment to catch its breath.
- ❌ **Liveness probe checking external DBs:** If your DB goes down, K8s restarts ALL your app pods! (Liveness should only check INTERNAL app health).
- ❌ **Too short `initialDelaySeconds`:** K8s kills your app before it finishes booting. (Fix: Use a Startup Probe!).

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between a Liveness probe and a Readiness probe?**
> "A Liveness probe checks if the container is deadlocked or frozen; if it fails, the kubelet restarts the container. A Readiness probe checks if the app is ready to serve traffic; if it fails, the pod is temporarily removed from the Service endpoints, but NOT restarted."

**Q: Why would you use a Startup probe?**
> "For legacy apps that have unpredictable, slow startup times (like a monolithic Java app taking 3 minutes). If we use a Liveness probe, it might kill the app before it boots. A Startup probe disables the other probes until it succeeds, giving the app time to wake up."

**Q: What happens if a Readiness probe fails?**
> "The pod's IP is immediately removed from the Endpoints object of any Service that targets it. Traffic stops flowing to that specific pod until the probe succeeds again. The pod is NOT restarted."

---

## ⚡ 30-Second Revision

- 🩺 **Liveness** = Restart me if I'm broken.
- 🛑 **Readiness** = Stop sending me traffic if I'm busy.
- ⏰ **Startup** = Give me extra time to boot up.
- 🛠️ 3 ways to check: HTTP (status 200), TCP (port open), Exec (exit 0).
- 🐛 Probes are the #1 cause of `CrashLoopBackOff` if misconfigured!
- 🔍 `kubectl describe pod` is how you debug failing probes.

> Next → [Resource Requests & Limits](03-resource-requests-limits.md) 🚀
