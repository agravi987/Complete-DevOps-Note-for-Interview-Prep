# 🏃‍♂️ Jobs & CronJobs

> **Hey Ravi! 👋** Up until now, everything we've built was designed to run FOREVER (like a web server). But what if you just want to run a database backup, or send a daily email, and then STOP? If a Deployment stops, it's considered a crash! For tasks that are meant to finish, we use **Jobs and CronJobs**. Let's get things done! ✅

---

## 🧠 What are Jobs and CronJobs?

> **Job:** Runs a task (Pod) until it successfully completes, then stops.
> **CronJob:** A Job that runs automatically on a schedule (like a Linux cron).

---

## 🧑‍🔧 The Hitman Analogy

- **Deployment (The Security Guard):** Expected to stand at the door forever. If they leave, something is wrong.
- **Job (The Hitman):** Given a specific mission. Goes in, does the job, reports "Mission Accomplished", and retires. 🎯
- **CronJob (The Mailman):** Has a set schedule. Shows up every day at 9 AM, delivers mail (does the job), and goes home. 📬

---

## 🏗️ Jobs (Run once to completion)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-job
spec:
  completions: 3        # I need this job to succeed 3 times total!
  parallelism: 2        # Run up to 2 pods at the exact same time
  backoffLimit: 4       # If it fails, retry up to 4 times before giving up
  template:
    spec:
      containers:
      - name: calculator
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never   # CRITICAL! Must be Never or OnFailure (not Always!)
```

### 🎛️ Job Knobs & Dials
- `completions`: How many total successful runs do I want?
- `parallelism`: How many pods can run simultaneously?
- `backoffLimit`: How many times K8s will retry if the pod crashes.
- `restartPolicy`: Unlike Deployments (which use `Always`), Jobs MUST use `Never` or `OnFailure`.

---

## ⏰ CronJobs (Run on a schedule)

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"     # Run every day at 2:00 AM!
  concurrencyPolicy: Forbid # If yesterday's job is still running, don't start today's!
  successfulJobsHistoryLimit: 3  # Keep the last 3 successful jobs (cleanup)
  failedJobsHistoryLimit: 1      # Keep the last 1 failed job
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: my-backup-script:v1
          restartPolicy: OnFailure
```

### 🗓️ The Cron Syntax (`* * * * *`)
1. Minute (0-59)
2. Hour (0-23)
3. Day of month (1-31)
4. Month (1-12)
5. Day of week (0-6) (Sunday=0)

*Example:* `0 2 * * *` = Minute 0, Hour 2 = 2:00 AM every day!

### 🚦 Concurrency Policy (What if jobs overlap?)
- `Allow` (Default): Run the new job anyway. Now you have 2 running!
- `Forbid`: Skip the new run. Let the old one finish.
- `Replace`: Kill the old running job and start the new one.

---

## 🎮 Commands

```bash
# Check Jobs
kubectl get jobs

# Check CronJobs
kubectl get cronjobs
kubectl get cj       # Short form!

# See the Pods created by the Job (they will say 'Completed')
kubectl get pods

# Read the logs of a completed job
kubectl logs math-job-abc12

# Trigger a CronJob MANUALLY right now (great for testing!)
kubectl create job --from=cronjob/daily-backup manual-backup-01
```

---

## 🗑️ Cleanup (TTL Controller)

By default, Jobs leave their "Completed" pods lying around so you can check their logs. But this clutters your cluster! You can tell K8s to auto-delete them:

```yaml
spec:
  ttlSecondsAfterFinished: 100   # Delete the job 100 seconds after it finishes!
```

---

## 🎤 Interview Knockout Answers

**Q: What is the difference between a Job and a Deployment?**
> "A Deployment is designed for long-running services (like web servers) and will infinitely restart pods to keep them running. A Job is for batch processing; it runs a pod until it successfully exits (exit code 0) and then considers the task complete."

**Q: Explain how `completions` and `parallelism` work in a Job.**
> "`completions` dictates the total number of successful pod executions required for the Job to be marked as complete. `parallelism` dictates how many of those pod executions can run concurrently at the exact same time."

**Q: What is concurrencyPolicy in a CronJob?**
> "It tells Kubernetes what to do if it's time to start a new job, but the previous scheduled job hasn't finished yet. 'Allow' runs them concurrently, 'Forbid' skips the new run, and 'Replace' kills the old job to start the new one."

---

## ⚡ 30-Second Revision

- 🏃‍♂️ **Job** = Run once, exit successfully, stop.
- ⏰ **CronJob** = Job on a schedule (cron syntax).
- 🔁 **restartPolicy** = Must be `Never` or `OnFailure` (never `Always`).
- 📈 `completions` = Total successful runs needed.
- 👯 `parallelism` = How many run at the same time.
- 🗑️ Use `ttlSecondsAfterFinished` or HistoryLimits to clean up old jobs!

> Next → [StatefulSets](06-statefulsets.md) 🚀
