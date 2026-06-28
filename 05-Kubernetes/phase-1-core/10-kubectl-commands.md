# ⌨️ kubectl Commands

Ravi, kubectl is your remote control for the cluster. 🎮 It helps you inspect, create, update, and debug resources without needing to stare at the dashboard all day.

## Simple Definition
kubectl Commands explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?
- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you the YAML and commands that matter in real life.

## Best-friend analogy
Think of kubectl like a remote control for the cluster.
- It lets you change the channel, pause, rewind, and inspect what is happening.
- It does not run the app itself; it talks to the API server.

## Technical explanation
- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture
- kubectl uses your kubeconfig.
- It sends requests to the API server.
- The API server returns the current cluster state.

## Workflow
1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram
`	ext
kubectl -> kubeconfig -> API Server -> Kubernetes Object
`

## Manifest example
`yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
spec:
  containers:
  - name: app
    image: nginx:1.27
`

Line by line:
- apiVersion tells Kubernetes which API family to use.
- kind tells Kubernetes what object you are creating.
- metadata.name gives the object a name.
- spec describes the desired state.
- The Pod is only here as a simple object to practice commands on.
- Most kubectl commands work by reading or changing this same desired state.

## kubectl commands
- kubectl get pods - list objects.
- kubectl describe pod <name> - inspect details.
- kubectl logs <name> - read logs.
- kubectl exec -it <name> -- sh - open a shell.
- kubectl apply -f file.yaml - create or update from YAML.
- kubectl delete -f file.yaml - remove an object.
- kubectl port-forward pod/<name> 8080:80 - test locally.
- kubectl explain pod - learn fields.

## File structure
No special structure is required, but many teams keep manifests in a manifests or k8s folder.

## Real production use cases
- Debugging live apps.
- Applying YAML during CI/CD.
- Inspecting cluster health quickly.

## Comparison table
| Imperative command | Declarative YAML |
| --- | --- |
| Tells Kubernetes what to do now | Tells Kubernetes what you want to exist |
| Good for quick fixes | Better for repeatable setups |

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
- What is the difference between kubectl get and kubectl describe?
- Why use kubectl apply instead of creating objects manually?
- What does kubectl exec do?

## Quick revision bullets
- kubectl Commands is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet
- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab
Practice get, describe, logs, exec, and apply on one Pod.

## Mini project
Create a small cheat sheet for your top five kubectl commands and use it in a mini cluster.

## Pro tips
- learn get, describe, logs, exec, apply first.
- kubectl explain is underrated and very helpful.

## Final summary
Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
