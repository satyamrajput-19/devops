# Day 29 — Kubernetes Basics

## Learning objectives
By the end of this lab, you should be able to:
- Explain Deployments and ReplicaSets and how labels/selectors connect resources.
- Deploy and scale Nginx replicas.
- Inspect rollout history, update an image, and roll back.
- Use ConfigMaps for non-sensitive configuration and Secrets for sensitive values.
- Configure readiness and liveness probes.
- Work with namespaces and ResourceQuotas.
- Demonstrate a simple blue/green traffic switch.
- Troubleshoot common Kubernetes workload issues.

> Run this in a disposable local Minikube cluster. These manifests are learning examples, not production-ready defaults. Verify every command and record actual output; do not claim a lab passed unless you ran it.

## 7-hour plan
1. **Hour 1:** Deployments, ReplicaSets, labels and selectors.
2. **Hour 2:** Scaling, rolling updates, rollout history and rollback.
3. **Hour 3:** ConfigMaps and Secrets.
4. **Hour 4:** Readiness and liveness probes.
5. **Hour 5:** Namespaces and ResourceQuotas.
6. **Hour 6:** Blue/green deployment and service selector switching.
7. **Hour 7:** Troubleshooting, interview practice, GitHub notes and cleanup.

## 1. Check your cluster

```bash
kubectl version --client
kubectl config current-context
kubectl get nodes
kubectl get namespaces
```

If using Minikube:

```bash
minikube status
minikube start --driver=docker
kubectl get nodes -o wide
```

If Docker is unavailable or Minikube fails, resolve that before continuing. Do not run commands against a production context.

## 2. Deployment and ReplicaSet

A **Deployment** manages desired application state and rollout history. It creates/manages **ReplicaSets**, which maintain the desired number of matching Pods. Pods are replaceable; do not treat a Pod name or IP as permanent.

Create `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - name: http
              containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 3
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 250m
              memory: 128Mi
```

Apply and inspect:

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get replicasets
kubectl get pods -l app=web -o wide
kubectl describe deployment web
kubectl rollout status deployment/web
```

**Important:** `spec.selector.matchLabels` must match the Pod template labels. For an `apps/v1` Deployment, its selector is immutable after creation.

### Labels and selectors
Labels are key/value metadata (for example, `app: web`). Selectors query resources with matching labels:

```bash
kubectl get pods --show-labels
kubectl get pods -l app=web
kubectl get pods -l 'app=web,tier=frontend'
```

The last command returns only Pods that have **both** labels; add `tier: frontend` to the template if you want it to match.

## 3. Scale, update, inspect, roll back

```bash
kubectl scale deployment/web --replicas=3
kubectl get pods -l app=web
kubectl set image deployment/web nginx=nginx:1.28
kubectl rollout status deployment/web
kubectl rollout history deployment/web
kubectl rollout undo deployment/web
kubectl rollout status deployment/web
```

A successful command does not by itself prove the workload is healthy. Check rollout status, Pod readiness, events and application behavior. In production, prefer reviewed manifest changes and pinned, verified image versions over ad-hoc imperative changes.

## 4. ConfigMap for non-sensitive configuration

Create a ConfigMap from the CLI:

```bash
kubectl create configmap web-settings --from-literal=APP_MODE=learning
kubectl get configmap web-settings -o yaml
```

Example Pod snippet to inject a ConfigMap value:

```yaml
env:
  - name: APP_MODE
    valueFrom:
      configMapKeyRef:
        name: web-settings
        key: APP_MODE
```

Place this under a container's `spec.containers[]`. ConfigMap updates consumed as environment variables do not automatically update the environment of already-running containers; restart/roll out workloads when needed.

## 5. Secret handling — never commit real credentials

For a disposable lab only, create a dummy Secret interactively:

```bash
kubectl create secret generic demo-credentials --from-literal=username=demo --from-literal=password='replace-me-locally'
kubectl get secret demo-credentials
```

To expose a Secret key as an environment variable:

```yaml
env:
  - name: DEMO_USERNAME
    valueFrom:
      secretKeyRef:
        name: demo-credentials
        key: username
```

**Security:** Kubernetes Secret data is base64-encoded in manifests/API representations, not encrypted merely because it is a Secret. Restrict RBAC, configure encryption at rest, avoid printing values in logs, and consider a dedicated secret manager for real workloads. Never put real credentials in Git, a public repository, or lesson screenshots.

## 6. Readiness vs liveness probes

- **Readiness probe:** decides whether a Pod should receive Service traffic.
- **Liveness probe:** detects a container that should be restarted.
- **Startup probe:** gives slow-starting applications time to start before liveness checks take effect.

For the Nginx example, the HTTP probe checks `/` on the named container port `http`. Probe thresholds and timing must be tuned for the real application; overly aggressive liveness checks can cause restart loops.

Inspect:

```bash
kubectl describe pod -l app=web
kubectl get pods -l app=web
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 7. Namespaces and ResourceQuota

```bash
kubectl create namespace day29-lab
kubectl get namespaces
kubectl get pods -n day29-lab
```

A namespace scopes names and many resource operations; it is not by itself a strong security boundary.

Create `quota.yaml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: lab-quota
  namespace: day29-lab
spec:
  hard:
    pods: "10"
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

Apply and inspect:

```bash
kubectl apply -f quota.yaml
kubectl describe resourcequota lab-quota -n day29-lab
```

Quota is enforced against namespace usage; a quota can reject new workloads if they exceed the remaining budget. Resource requests/limits and quotas should be designed together.

## 8. Blue/green deployment exercise

Blue/green uses two separately labeled versions and a Service selector to direct traffic to one version at a time. This is a simplified local exercise; it does not automatically provide database compatibility, session handling, or a complete production release process.

Create two Deployments:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      version: blue
  template:
    metadata:
      labels:
        app: web
        version: blue
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-green
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      version: green
  template:
    metadata:
      labels:
        app: web
        version: green
    spec:
      containers:
        - name: nginx
          image: nginx:1.28
          ports:
            - containerPort: 80
```

Save as `blue-green.yaml`, then apply:

```bash
kubectl apply -f blue-green.yaml
kubectl rollout status deployment/web-blue
kubectl rollout status deployment/web-green
```

Create `web-traffic-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-traffic
spec:
  type: ClusterIP
  selector:
    app: web
    version: blue
  ports:
    - port: 80
      targetPort: 80
```

Switch to green after verifying it is healthy:

```bash
kubectl apply -f web-traffic-service.yaml
kubectl get endpoints web-traffic
kubectl patch service web-traffic -p '{"spec":{"selector":{"app":"web","version":"green"}}}'
kubectl get endpoints web-traffic
```

To switch back to blue:

```bash
kubectl patch service web-traffic -p '{"spec":{"selector":{"app":"web","version":"blue"}}}'
```

The Service selector switch is the traffic change. Both versions should be ready before switching. Confirm the selected endpoints and test the app via port-forward if desired:

```bash
kubectl port-forward service/web-traffic 8080:80
```

In another terminal, use `curl -I http://127.0.0.1:8080`. Since both images are Nginx, the response alone may not identify the version; use distinguishable app versions/content for a meaningful real-world exercise.

## Troubleshooting quick guide

| Symptom | What to inspect | Likely causes / next step |
|---|---|---|
| Deployment not ready | `kubectl describe deployment web` | Selector mismatch, scheduling or image issue |
| Pod Pending | `kubectl describe pod <pod>` | Insufficient resources, quota, scheduling constraints |
| ImagePullBackOff | Pod events | Invalid tag, registry access or image pull credentials |
| CrashLoopBackOff | `kubectl logs <pod> --previous` | Process exits, bad config, failed startup |
| Probe failures | `kubectl describe pod <pod>` | Wrong path/port, slow startup, app unhealthy |
| Service has no endpoints | `kubectl get endpoints web-traffic` | Service selector does not match ready Pod labels |
| Quota exceeded | `kubectl describe resourcequota -n day29-lab` | Namespace hard limit reached; inspect requests/limits |

Use `kubectl get events --sort-by=.metadata.creationTimestamp` and `kubectl describe` before guessing.

## Interview questions and short answers

1. **Deployment vs ReplicaSet?** Deployment manages declarative updates and rollback history; ReplicaSet maintains the requested number of matching Pods.
2. **What causes a Service to have no endpoints?** Often selector/label mismatch, no matching Pods, or matching Pods not ready.
3. **Readiness vs liveness?** Readiness controls traffic eligibility; liveness can trigger container restart.
4. **ConfigMap vs Secret?** Both supply configuration; ConfigMap is for non-sensitive data, Secret is intended for sensitive values but requires access controls and encryption safeguards.
5. **What is a rollout rollback?** Reverting a Deployment to a prior revision with `kubectl rollout undo`, subject to available history.
6. **What does a namespace provide?** A logical scope for names and resource policies, not a standalone security boundary.
7. **Blue/green vs rolling update?** Blue/green keeps two versions available and switches traffic; rolling update gradually replaces replicas under a Deployment strategy.

## Cleanup

If you used these exact resources:

```bash
kubectl delete service web-traffic --ignore-not-found
kubectl delete -f blue-green.yaml --ignore-not-found
kubectl delete -f deployment.yaml --ignore-not-found
kubectl delete configmap web-settings --ignore-not-found
kubectl delete secret demo-credentials --ignore-not-found
kubectl delete -f quota.yaml --ignore-not-found
kubectl delete namespace day29-lab --ignore-not-found
```

## Day 29 completion checklist
- [ ] Explained Deployment, ReplicaSet, Pod, label and selector.
- [ ] Deployed and inspected two replicas.
- [ ] Scaled, updated, inspected rollout history and rolled back.
- [ ] Created a ConfigMap and a dummy Secret without committing credentials.
- [ ] Inspected readiness/liveness probe status.
- [ ] Created a namespace and ResourceQuota.
- [ ] Switched a Service between blue and green Deployments and checked endpoints.
- [ ] Recorded actual command output, errors and fixes in GitHub.
- [ ] Cleaned up lab resources.

**Next:** Day 30 — Kubernetes on AWS (EKS) & App Operations: Terraform-based EKS provisioning, node groups, IRSA, PV/PVC, auto-healing, HPA, rollout strategies and External Secrets Operator.
