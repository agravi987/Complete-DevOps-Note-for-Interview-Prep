# 🧾 ConfigMaps

Ravi, a ConfigMap is like a notebook of settings for your app. 📓 It keeps non-sensitive configuration outside the image so you can change it easily. No rebuild needed, just update the note. ✍️

## Simple Definition

ConfigMaps explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?

- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you the YAML and commands that matter in real life.

## Best-friend analogy

Think of a ConfigMap like a sticky note with app settings.

- It holds plain text config.
- You can reuse the same image in different environments.

## Technical explanation

- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture

- ConfigMap stores key-value data.
- Pods can read it as environment variables or mounted files.
- The app image stays unchanged.

## Workflow

1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram

`	ext
ConfigMap -> Pod env vars or mounted file -> Application
`

## Manifest example

`yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: production
  LOG_LEVEL: info
`

Line by line:

- apiVersion tells Kubernetes which API family to use.
- kind tells Kubernetes what object you are creating.
- metadata.name gives the object a name.
- spec describes the desired state.
- `data` stores key-value pairs.
- The same ConfigMap can be mounted as a file or injected as environment variables.

## kubectl commands

- kubectl get configmap - list ConfigMaps.
- kubectl describe configmap app-config - inspect values.
- kubectl exec -it <pod> -- printenv - check environment values inside a Pod.
- kubectl create configmap app-config --from-literal=APP_MODE=production - create one quickly.

## File structure

A common setup is configmap.yaml plus a deployment.yaml that consumes it.

## Real production use cases

- Different config for dev, staging, and production.
- Feature flags and log levels.
- Reusing one image in many environments.

## Comparison table

| ConfigMap                     | Secret                              |
| ----------------------------- | ----------------------------------- |
| Plain text config             | Sensitive data                      |
| Good for non-sensitive values | Use for passwords, tokens, and keys |

## Common mistakes

- Forgetting that Kubernetes follows desired state.
- Changing the wrong field in YAML.
- Ignoring events when troubleshooting.

## Best practices

1. Keep manifests small and readable.
2. Use one clear label pattern.
3. Check kubectl describe when something feels off.

## Troubleshooting guide

- If something does not start, read kubectl describe.
- If you need app output, read kubectl logs.
- If traffic does not flow, check selectors and endpoints.

## Top interview questions

- What problem does a ConfigMap solve?
- How does a Pod read a ConfigMap?
- Why not put config directly in the image?

## Quick revision bullets

- ConfigMaps is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet

- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab

Create a ConfigMap and inject it into a Pod as environment variables.

## Mini project

Run the same app image in dev and prod with different ConfigMaps.

## Pro tips

- Keep secrets out of ConfigMaps.
- Treat config as data, not as a baked-in image change.

## Final summary

Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
