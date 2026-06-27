# 🚀 Kubernetes Deployments

## 🖼️ Quick Visual Summary

![Quick Summary: Kubernetes Deployments](../assets/topic-summaries/kubernetes-deployments.svg)

> **80/20 Summary:** a Deployment manages ReplicaSets, versions your app, and gives you safe rollouts and rollbacks. 🔁

## 1. Big Picture

Ravi, this is the one you will use in real work.

Direct Pods are too fragile for production.
A Deployment gives you a safe way to release a new version, scale the app, and roll back when things go wrong.

## 2. Real-Life Analogy

Think of a Deployment like a restaurant manager changing the menu while guests are still eating 🍽️

- old dishes stay available until the new dish is ready
- new dishes are tested before serving everyone
- if the new recipe fails, the manager goes back to the old menu

## 3. Technical Definition

A Deployment is a Kubernetes controller that manages ReplicaSets and provides declarative updates, scaling, and rollbacks for stateless applications.

## 4. Internal Working

```text
Deployment YAML changed
   |
   v
New ReplicaSet created
   |
   | scale up new Pods
   | scale down old Pods
   v
Readiness probe passes
   |
   v
Traffic moves safely
```

## 5. Key Concepts

| Concept | Meaning |
| --- | --- |
| Deployment | High-level object for app lifecycle 🔁 |
| ReplicaSet | The versioned Pod manager behind the scenes 📦 |
| Rollout | The process of moving to a new version 🚦 |
| Rollback | Return to a previous working version ⏪ |
| `RollingUpdate` | Default strategy that replaces Pods gradually 🧊 |
| `Recreate` | Stops old Pods first, then starts new ones ⚠️ |
| Readiness probe | Tells Kubernetes when a Pod can receive traffic ✅ |

## 6. Commands

| Command | Why we use it | What happens internally |
| --- | --- | --- |
| `kubectl apply -f deployment.yaml` | Create or update a Deployment | Sends desired state to the API server |
| `kubectl rollout status deployment/<name>` | Watch release progress | Checks rollout conditions until complete |
| `kubectl rollout history deployment/<name>` | See past revisions | Reads Deployment revision history |
| `kubectl rollout undo deployment/<name>` | Roll back a bad release | Switches back to an older ReplicaSet |
| `kubectl scale deployment/<name> --replicas=6` | Change app size | Updates desired replica count |
| `kubectl set image deployment/<name> ...` | Update image quickly | Changes the Pod template and triggers a new rollout |

## 7. Real Production Usage

Teams use Deployments for:

- frontend releases
- backend API releases
- worker version upgrades
- config-driven app changes

In practice, a Deployment is often connected to:

- a Service for traffic
- a ConfigMap or Secret for configuration
- probes for health checks
- CI/CD pipelines for automated rollout

## 8. Common Mistakes

- ❌ Deploying raw Pods in production
  - Why it is wrong: they have no built-in self-management.
  - ✅ Correct: use a Deployment.

- ❌ Using `latest` for release images
  - Why it is wrong: it makes rollbacks and change tracking unclear.
  - ✅ Correct: use versioned tags or Git SHAs.

- ❌ Skipping readiness probes
  - Why it is wrong: traffic may reach an app before it is ready.
  - ✅ Correct: gate traffic with a readiness probe.

## 9. Best Practices

1. Use Deployments for production workloads.
2. Keep rollout changes small.
3. Use specific image versions.
4. Add readiness probes.
5. Monitor rollout status during releases.

## 10. Interview Corner

Ravi, your interviewer might ask this:

**Q1: Why use a Deployment instead of a ReplicaSet?**
A1: Deployments add rollout and rollback support.

**Q2: What is a rollout?**
A2: The process of moving from one app version to another.

**Q3: What is rollback?**
A3: Returning to a previous stable revision.

**Q4: Why are readiness probes important?**
A4: They stop traffic from reaching an app that is not ready yet.

**Q5: What is the default Deployment strategy?**
A5: `RollingUpdate`.

## 11. Revision Summary

- Deployment manages the app lifecycle.
- ReplicaSets sit underneath it.
- Rollouts are gradual.
- Rollbacks protect you from bad releases.

## 12. Key Takeaways

- Use Deployments for stateless apps.
- Never trust a release without readiness checks.
- Rollback is part of the release plan.

## 13. Comparison Table

| Deployment | ReplicaSet |
| --- | --- |
| Manages versions and rollouts | Manages replica count |
| Preferred for production | Usually managed by Deployment |
| Supports rollback | No built-in release history |

## 14. Memory Tricks

- **Deployment = release manager**
- **Rollout = moving traffic safely**
- **Rollback = undo button**

## 15. Official Docs

- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rollouts](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)
