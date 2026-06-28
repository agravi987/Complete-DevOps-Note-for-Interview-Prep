# 🔄 Rolling Updates

## Quick Visual Summary

> **80/20 Summary:** change gradually, keep old Pods alive, and keep rollback ready.

Ravi, rolling updates are how Kubernetes upgrades your app without making everyone stare at a downtime screen. 😄 It swaps Pods gradually and keeps the service alive like a smooth elevator instead of a broken staircase. 🛗

## Simple Definition

Rolling Updates explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?

- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you the YAML and commands that matter in real life.

## Best-friend analogy

Think of changing seats in a theater while the movie is still playing.

- You do not stop the movie for everyone.
- You replace seats one row at a time.
- If the new row has a problem, the old row still exists.

## Technical explanation

- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture

- The Deployment owns the rollout.
- Old and new ReplicaSets exist together for a while.
- Readiness probes decide when traffic can move.

## Workflow

1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram

`	ext
Old ReplicaSet <--> New ReplicaSet
      |                   |
   scale down          scale up
      |                   |
     old Pods          new Pods
`

## Manifest example

`yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-demo
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: rolling-demo
  template:
    metadata:
      labels:
        app: rolling-demo
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
- `type: RollingUpdate` is the default update strategy.
- `maxSurge` allows extra Pods during the rollout.
- `maxUnavailable` limits how many Pods can go offline at once.

## kubectl commands

- kubectl rollout status deployment/rolling-demo - watch progress.
- kubectl rollout history deployment/rolling-demo - see revisions.
- kubectl rollout undo deployment/rolling-demo - go back to a working version.
- kubectl rollout pause deployment/rolling-demo - pause a release if needed.
- kubectl rollout resume deployment/rolling-demo - continue the rollout.

## File structure

Usually one deployment.yaml file is enough for the rollout example.

## Real production use cases

- Safe application upgrades.
- Zero-downtime release pipelines.
- Version-by-version production releases.

## Comparison table

| RollingUpdate                               | Recreate                    |
| ------------------------------------------- | --------------------------- |
| Keeps old Pods running while new ones start | Stops old Pods first        |
| Safer for most apps                         | Simpler but causes downtime |

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

- What is a rolling update?
- Why are readiness probes important during rollout?
- What do maxSurge and maxUnavailable do?

## Quick revision bullets

- Rolling Updates is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet

- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab

Change the image tag in a Deployment and watch the rollout in real time.

## Mini project

Set up a Deployment release flow where you can roll back with one command.

## Pro tips

- Small releases are easier to trust and easier to undo.
- Watch the rollout before you walk away.

## Final summary

Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
