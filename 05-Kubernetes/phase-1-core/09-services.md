# 🌐 Services

## Quick Visual Summary

![Quick Summary: Kubernetes Services and Networking](../../assets/topic-summaries/kubernetes-services-networking.svg)

> **80/20 Summary:** Pods are temporary, Services are stable, and kube-proxy sends traffic to the right Pod.

Ravi, a Service is the friendly front desk for your Pods. 🛎️ It gives traffic a stable address even when the Pods behind it keep changing. Think of it as the reliable phone number for a moving team. 📞

## Simple Definition

Services explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?

- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you the YAML and commands that matter in real life.

## Best-friend analogy

Think of a Service like a hotel front desk.

- Guests ask for the hotel, not a random room.
- The desk knows which rooms are open.
- If a room changes, the desk still stays the same.

## Technical explanation

- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture

- A Service selects Pods using labels.
- DNS gives the Service a name inside the cluster.
- kube-proxy helps route traffic to the right endpoint.

## Workflow

1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram

`	ext
Client -> Service DNS -> Service IP -> kube-proxy -> Pod IPs
`

## Manifest example

`yaml
apiVersion: v1
kind: Service
metadata:
name: web-service
spec:
type: ClusterIP
selector:
app: web
ports:

- port: 80
  targetPort: 8080
  `

Line by line:

- apiVersion tells Kubernetes which API family to use.
- kind tells Kubernetes what object you are creating.
- metadata.name gives the object a name.
- spec describes the desired state.
- `type: ClusterIP` keeps the Service internal.
- `selector` matches the Pods that should receive traffic.
- `port` is the Service port and `targetPort` is the container port.

## kubectl commands

- kubectl get svc - see Services and cluster IPs.
- kubectl describe svc web-service - inspect ports and selectors.
- kubectl get endpoints web-service - verify which Pods are behind it.

## File structure

Common files are deployment.yaml and service.yaml, one for the app and one for the stable front door.

## Real production use cases

- Internal APIs.
- Database access inside the cluster.
- Stable access to microservices.

## Comparison table

| ClusterIP     | NodePort                     | LoadBalancer               | ExternalName                 |
| ------------- | ---------------------------- | -------------------------- | ---------------------------- |
| Internal only | Exposes a port on every node | Uses a cloud load balancer | Maps to an external DNS name |
| Most common   | Simple testing and exposure  | Public cloud apps          | DNS aliasing                 |

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

- Why do Kubernetes apps need Services?
- What is the difference between port and targetPort?
- Why is ClusterIP the default Service type?

## Quick revision bullets

- Services is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet

- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab

Create a Deployment and a Service, then reach the app through the Service name.

## Mini project

Expose a small Nginx app using ClusterIP and explain how traffic reaches the Pod.

## Pro tips

- Always think in terms of stable front door and temporary backends.
- Check endpoints whenever a Service looks broken.

## Final summary

Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
