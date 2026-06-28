# 🔐 Secrets

Ravi, Secrets are the locked drawer of Kubernetes. 🗝️ They hold passwords, tokens, and certificates carefully so you do not expose them casually.

## Simple Definition
Secrets explains how to solve one real Kubernetes problem in a practical way.

## Why do we need this?
- Kubernetes feels much easier when you learn one clear problem at a time.
- This topic shows you the YAML and commands that matter in real life.

## Best-friend analogy
Think of a Secret like a locked drawer.
- It is still stored in the cluster, but it is meant for sensitive data.
- You should keep access tight and avoid exposing it casually.

## Technical explanation
- Beginner: learn the basic idea and what it solves.
- Intermediate: connect the object to the controller or node that uses it.
- Advanced: understand how Kubernetes keeps desired state and actual state in sync.

## Internal architecture
- Secret holds sensitive key-value data.
- Pods can read it as environment variables or mounted files.
- Base64 encoding is used for transport, not as real encryption by itself.

## Workflow
1. You write YAML.
2. kubectl sends it to the API server.
3. Kubernetes stores and reconciles the desired state.
4. The cluster makes reality match the YAML.

## ASCII diagram
`	ext
Secret -> Pod env vars or mounted file -> Application
`

## Manifest example
`yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_USER: admin
  DB_PASSWORD: strong-password
`

Line by line:
- apiVersion tells Kubernetes which API family to use.
- kind tells Kubernetes what object you are creating.
- metadata.name gives the object a name.
- spec describes the desired state.
- `type: Opaque` is the most common generic Secret type.
- `stringData` lets you write plain text and Kubernetes stores it appropriately.

## kubectl commands
- kubectl get secret - list Secrets.
- kubectl describe secret app-secret - inspect the object metadata.
- kubectl get secret app-secret -o yaml - see the encoded data.
- echo c3VwZXJzZWNyZXQ= | base64 -d - decode a sample value.

## File structure
A common setup is secret.yaml plus deployment.yaml, and the Secret is never committed as plain text in a public repo.

## Real production use cases
- Database usernames and passwords.
- API tokens and certificates.
- Any sensitive value that your app needs at runtime.

## Comparison table
| Secret | ConfigMap |
| --- | --- |
| Sensitive data | Non-sensitive config |
| Keep access restricted | Safe for plain text settings |

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
- What is a Secret in Kubernetes?
- Is base64 the same as encryption?
- Why should Secrets be protected more carefully than ConfigMaps?

## Quick revision bullets
- Secrets is about solving one real Kubernetes problem.
- YAML declares the desired state.
- kubectl is how you observe and control it.

## One-page cheat sheet
- kubectl get ...
- kubectl describe ...
- kubectl apply -f ...

## Hands-on lab
Create a Secret and mount it into a Pod, then verify the app can read it.

## Mini project
Build a small app that reads database credentials from a Secret.

## Pro tips
- Base64 is not encryption, so treat Secret access seriously.
- Do not print Secret values in logs unless you are testing locally.

## Final summary
Ravi, this topic is useful because it connects the problem, the manifest, and the commands into one simple mental model.
