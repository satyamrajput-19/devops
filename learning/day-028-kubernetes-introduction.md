# Day 28 — Kubernetes Introduction

## Today's objective

Session 28 introduces container orchestration and Kubernetes architecture, including nodes, Pods, Services, kubectl, and Minikube. The goal is to deploy your first Pod and Service in a local Kubernetes lab.

## 7-hour study plan

| Time | Focus | Deliverable |
|---|---|---|
| Hour 1 | Why orchestration? Kubernetes architecture | Explain control plane, worker nodes, Pods |
| Hour 2 | Install/check kubectl and Minikube | Working local cluster |
| Hour 3 | Inspect the cluster | Nodes, namespaces, and system Pods |
| Hour 4 | Deploy your first Pod | A running Nginx Pod |
| Hour 5 | Expose it with a Service | Reach the application locally |
| Hour 6 | Troubleshooting and cleanup | Inspect events, logs, and resources |
| Hour 7 | GitHub documentation and interviews | Commit lesson and verified lab results |

## 1. Why Kubernetes?

Containers package an application and its dependencies. Kubernetes orchestrates containers across a cluster and helps manage deployment, networking, desired state, and recovery.

Key concepts:

- **Cluster:** the Kubernetes environment.
- **Control plane:** makes cluster-level decisions and exposes the Kubernetes API. Main components include the API server, scheduler, controller manager, and usually etcd.
- **Worker node:** runs application workloads; typically includes kubelet, a container runtime, and networking components.
- **Pod:** the smallest deployable Kubernetes unit; it can contain one or more closely related containers.
- **Service:** provides a stable network endpoint for a set of Pods selected by labels.
- **kubectl:** command-line tool for interacting with the Kubernetes API.
- **Minikube:** runs a local Kubernetes cluster for learning and development.

## 2. Install and start Minikube

You need a compatible container runtime or VM driver (Docker is one common option), plus kubectl and Minikube. Follow the official installation instructions for your OS; commands and driver availability can differ.

Check installations:

```bash
kubectl version --client
minikube version
docker --version
```

Start the local cluster using a driver installed on your machine:

```bash
minikube start --driver=docker
```

If Docker is not installed or running, choose a supported driver for your system instead. Verify the cluster:

```bash
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
kubectl get pods -A
```

The node should eventually show `Ready`. A local cluster can take several minutes to download images and start system components.

## 3. Deploy your first Pod

Create `nginx-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80
```

Apply and inspect it:

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods -o wide
kubectl describe pod nginx-pod
kubectl logs nginx-pod
```

Wait for the Pod to become Ready. If it remains Pending or shows an image-pull error, inspect the events in `kubectl describe pod nginx-pod`.

This single-Pod manifest is useful for learning. In production, a Deployment is generally preferred for managing replicated stateless application Pods; you'll study Deployments in the next Kubernetes session.

## 4. Create a Service

A Pod's IP can change when the Pod is replaced. A Service gives a stable endpoint and selects Pods using labels.

Create `nginx-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
```

Apply and inspect:

```bash
kubectl apply -f nginx-service.yaml
kubectl get services
kubectl get endpointslices
```

The Service selector `app: nginx` must match the Pod label. If it does not, the Service may have no usable endpoints.

## 5. Access the application locally

A ClusterIP Service is reachable inside the cluster. Use port forwarding to access it from your own machine:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Keep that command running in one terminal. In another terminal, request the local endpoint:

```bash
curl -I http://127.0.0.1:8080
```

Stop port-forwarding with Ctrl+C. This method avoids exposing the service publicly.

## 6. Essential kubectl commands

```bash
kubectl get nodes
kubectl get pods
kubectl get pods -o wide
kubectl get services
kubectl describe pod nginx-pod
kubectl logs nginx-pod
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl explain pod
kubectl delete -f nginx-service.yaml
kubectl delete -f nginx-pod.yaml
```

Useful distinctions:

- `get`: lists resources.
- `describe`: shows resource details and events.
- `logs`: displays container logs.
- `apply`: creates or updates resources from declarative YAML.
- `delete -f`: removes resources declared in a file.

## 7. Troubleshooting

| Symptom | Checks |
|---|---|
| Minikube will not start | Driver installed, Docker running, available CPU/memory, Minikube logs |
| Node NotReady | `kubectl describe node`, system Pods, local cluster status |
| Pod Pending | `kubectl describe pod`, scheduling events and available resources |
| ImagePullBackOff | Image name/tag, registry reachability, image pull permissions |
| CrashLoopBackOff | `kubectl logs`, previous logs with `--previous`, container command and configuration |
| Service has no endpoints | Selector matches Pod labels; Pod is running and ready |
| Port-forward fails | Service name/port, Pod status, port already in use |
| YAML apply fails | Indentation, API version, resource kind, field names |

## 8. Interview questions

1. What problem does Kubernetes solve?
2. What are the main control-plane components?
3. What is a worker node?
4. What is a Pod, and why is it the smallest deployable unit?
5. What is the difference between a Pod and a container?
6. Why do we use a Service?
7. What is the difference between ClusterIP, NodePort, and LoadBalancer Services?
8. What is kubectl?
9. What is Minikube used for?
10. How would you troubleshoot a Pod stuck in Pending or ImagePullBackOff?

## 9. Cleanup

When you're done, delete the lab resources:

```bash
kubectl delete -f nginx-service.yaml
kubectl delete -f nginx-pod.yaml
```

To stop Minikube while keeping its cluster data:

```bash
minikube stop
```

To delete the local cluster entirely (this removes its local cluster resources), use:

```bash
minikube delete
```

## 10. Completion checklist

- [ ] Install/check kubectl and Minikube.
- [ ] Start a local Kubernetes cluster.
- [ ] Inspect nodes, namespaces, and system Pods.
- [ ] Create and apply an Nginx Pod manifest.
- [ ] Inspect the Pod with get, describe, and logs.
- [ ] Create a ClusterIP Service matching the Pod label.
- [ ] Access Nginx through port-forwarding.
- [ ] Troubleshoot at least one issue or explain the diagnostic steps.
- [ ] Clean up lab resources.
- [ ] Commit the lesson and actual lab results to GitHub.

**Next:** Day 29 — Kubernetes Basics: Deployments, ReplicaSets, ConfigMaps, Secrets, labels/selectors, rollouts, probes, namespaces, and blue/green deployment concepts.
