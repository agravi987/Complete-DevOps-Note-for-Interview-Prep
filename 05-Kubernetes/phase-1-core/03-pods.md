# Pods
## Quick Visual Summary

![Quick Summary: Pods and ReplicaSets](../../assets/topic-summaries/pods-replicasets.svg)

> **80/20 Summary:** Pods run the work, and higher-level controllers keep them alive.

Ravi, keep this note simple. A Pod is the smallest unit Kubernetes actually runs.

## Simple Definition
Pods explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?
- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you how to write the YAML and use the commands that matter in real life.

## Best-friend analogy
Think of a Pod like a small apartment room.
- The containers inside the room share the same address.
- They can talk through `localhost`.
- If the room is gone, Kubernetes makes a new room.

## Technical explanation
- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture
- Pod IP is shared by all containers inside the Pod.
- Each container keeps its own process space.
- Shared volumes let containers exchange files.

## Workflow
1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram
`	ext
Pod
|-- container A
|-- container B
|-- shared network
|-- shared volumes
`

## Manifest example
`yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
spec:
  containers:
  - name: app
    image: nginx:1.27
`

Line by line:
- piVersion tells Kubernetes which API family to use.
- kind tells Kubernetes what object you are creating.
- metadata.name gives the object a name.
- spec describes the desired state.
- `kind: Pod` says this is the workload unit Kubernetes runs.
- `containers` lists what runs inside the Pod.

## kubectl commands
- `kubectl get pods` - list Pods.
- `kubectl describe pod <name>` - inspect status and events.
- `kubectl logs <name>` - read container logs.
- `kubectl exec -it <name> -- sh` - open a shell inside the Pod.

## File structure
Usually one `pod.yaml` is enough while learning.

## Real production use cases
- Running a single web app container.
- Running a sidecar pair like app + log shipper.
- Running helper containers for jobs or diagnostics.

## Comparison table
| Pod | Container |
| --- | --- |
| Kubernetes unit | Runtime unit |
| Can hold multiple containers | One process inside the Pod |
| Shares network and storage | Owns a process in that Pod |

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
- What is a Pod?
- Why do containers inside one Pod share the same IP?
- Why should we not run unrelated apps in one Pod?

## Quick revision bullets
- Pods is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet
- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab
Create a Pod, check its IP, then delete it and notice that the IP is temporary.

## Mini project
Deploy a single Nginx Pod and practice logs, exec, and describe.

## Pro tips
- Think of a Pod as a wrapper around one app or a tiny app team.
- Most real systems use Pods through Deployments, not directly.

## Final summary
Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
