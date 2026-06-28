# 🧩 Kubernetes Components

Ravi, this is the cast list of the Kubernetes show. 🎭 Each component has a job, and together they make the whole cluster function.

## Simple Definition

Kubernetes Components explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?

- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you how to write the YAML and use the commands that matter in real life.

## Best-friend analogy

Think of Kubernetes like a restaurant.

- The API server is the front desk.
- The scheduler is the host who assigns a table.
- The controller manager is the manager who keeps the restaurant stable.
- `etcd` is the notebook with the truth.
- The kubelet is the waiter on each table.
- `kube-proxy` helps route customers to the right place.

## Technical explanation

- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture

- API server: entry point for all requests.
- etcd: stores cluster state.
- scheduler: picks the node.
- controller manager: keeps desired state true.
- kubelet: runs on each node.
- kube-proxy: routes traffic on the node.

## Workflow

1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram

`	ext
kubectl
  |
API Server
  |--- etcd
  |--- Scheduler
  |--- Controller Manager
  |
Worker Node
  |--- kubelet
  |--- kube-proxy
  |--- container runtime
`

## Manifest example

`yaml
apiVersion: v1
kind: Pod
metadata:
name: components-demo
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
- A small Pod is enough to show how the components work together.
- In managed Kubernetes, many control plane components are hidden from you.

## kubectl commands

- `kubectl get nodes` - check worker node health.
- `kubectl get pods -n kube-system` - see core system Pods in managed clusters.
- `kubectl describe pod <pod>` - inspect kubelet events and container status.

## File structure

No special file structure. Keep one YAML file for each experiment while learning.

## Real production use cases

- Managed clusters on EKS, GKE, and AKS.
- Platform teams debugging scheduling or node issues.
- SRE teams checking control plane health.

## Comparison table

| Control plane components                 | Node components              |
| ---------------------------------------- | ---------------------------- |
| Decide and store state                   | Run the Pod workload         |
| API server, scheduler, controllers, etcd | kubelet, kube-proxy, runtime |

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

- What does the API server do?
- What is the kubelet responsible for?
- Why is `etcd` important?

## Quick revision bullets

- Kubernetes Components is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet

- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab

Describe a Pod and map each status message back to the component that produced it.

## Mini project

Draw the request path from `kubectl apply` to a running Pod and explain each hop.

## Pro tips

- If you know which component owns a problem, debugging becomes much faster.
- Most interview answers get stronger when you explain the job and the location of each component.

## Final summary

Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
