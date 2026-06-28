# Namespaces


Ravi, keep this note simple. Namespaces split one cluster into logical spaces so teams, apps, and environments stay organized.

## Simple Definition
Namespaces explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?
- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you the YAML and commands that matter in real life.

## Best-friend analogy
Think of namespaces like separate rooms in one building.
- Dev team in one room.
- QA team in another room.
- Everyone shares the building, but the rooms keep things tidy.

## Technical explanation
- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture
- Namespace is a logical grouping, not a separate cluster.
- Resources inside a namespace share that namespace scope.
- Good for separation, quotas, and clean organization.

## Workflow
1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram
`	ext
Cluster
|-- namespace dev
|   |-- pods
|   |-- services
|-- namespace prod
    |-- pods
    |-- services
`

## Manifest example
`yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
`

Line by line:
- apiVersion tells Kubernetes which API family to use.
- kind tells Kubernetes what object you are creating.
- metadata.name gives the object a name.
- spec describes the desired state.
- Namespace names group resources.
- Use one namespace per environment or team when it helps reduce confusion.

## kubectl commands
- kubectl get namespaces - list namespaces.
- kubectl create namespace dev - create one fast.
- kubectl config set-context --current --namespace=dev - switch the default namespace.

## File structure
A common setup is to keep one folder per environment, such as manifests/dev and manifests/prod.

## Real production use cases
- Separate dev, test, and prod resources.
- Give teams their own working area.
- Apply resource quotas and policies by environment.

## Comparison table
| Namespace | Label |
| --- | --- |
| Groups resources by scope | Tags resources for filtering |
| Affects default visibility | Helps search and selection |

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
- What is a namespace?
- Is a namespace a security boundary by itself?
- Why do teams use namespaces in real projects?

## Quick revision bullets
- Namespaces is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet
- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab
Create two namespaces and deploy the same app into each one.

## Mini project
Set up dev and prod namespaces and explain how you would organize resources in each.

## Pro tips
- Namespaces help with organization first, not magic security.
- Combine namespaces with RBAC and quotas for better control.

## Final summary
Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
