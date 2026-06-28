# 📄 YAML Manifest Files

Ravi, YAML is the language you use to speak to Kubernetes. 🗣️ If you write the manifest clearly, the cluster understands your intention.

## Simple Definition
YAML Manifest Files explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?
- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you the YAML and commands that matter in real life.

## Best-friend analogy
Think of YAML like a recipe.
- Ingredients are the fields.
- The steps are the spec.
- If the structure is wrong, the recipe fails.

## Technical explanation
- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture
- apiVersion picks the API group.
- kind picks the resource type.
- metadata holds names and labels.
- spec holds the desired state.

## Workflow
1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram
`	ext
YAML file -> kubectl apply -> API Server -> running object
`

## Manifest example
`yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yaml-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: yaml-demo
  template:
    metadata:
      labels:
        app: yaml-demo
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
- `apiVersion` and `kind` must match the resource you want.
- `metadata` names the object and can hold labels.
- `spec` tells Kubernetes what should exist.

## kubectl commands
- kubectl apply -f file.yaml - create or update.
- kubectl diff -f file.yaml - preview changes.
- kubectl explain deployment.spec - learn fields.
- kubectl get deployment yaml-demo -o yaml - inspect live YAML.

## File structure
A simple structure is:
- manifests/deployment.yaml
- manifests/service.yaml
- manifests/configmap.yaml

## Real production use cases
- All Kubernetes resources start as YAML.
- CI/CD pipelines apply YAML files to clusters.
- Teams keep infrastructure in Git using YAML.

## Comparison table
| YAML | JSON |
| --- | --- |
| Easier to read and write | More machine friendly |
| Common in Kubernetes | Supported too, but less common for hand-written manifests |

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
- What are apiVersion, kind, metadata, and spec?
- Why is indentation so important in YAML?
- Why do teams prefer YAML for Kubernetes?

## Quick revision bullets
- YAML Manifest Files is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet
- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab
Write one Deployment manifest and use kubectl explain to understand each field.

## Mini project
Create a small manifests folder with a Deployment, Service, and ConfigMap.

## Pro tips
- Keep YAML short and readable.
- Indentation mistakes are the most common beginner problem.

## Final summary
Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
