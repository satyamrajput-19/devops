# Day 30 — Kubernetes on AWS (EKS) & App Operations

## Session scope
This lesson follows Session 30 of the DevOps & Cloud Engineering course:
- Provision EKS with Terraform.
- Understand managed node groups and IRSA.
- Work with PersistentVolumes (PV) and PersistentVolumeClaims (PVC).
- Learn auto-healing, Horizontal Pod Autoscaler (HPA), and rollout strategies.
- Understand secrets synchronization with External Secrets Operator (ESO).

> **Cost and safety warning:** EKS, worker nodes, load balancers, NAT gateways, EBS volumes, and related resources can incur AWS charges even when your app is idle. Prefer diagrams, local Kubernetes, Terraform validation/planning, or an approved sandbox first. Do not run `terraform apply` against a paid AWS account until you understand the plan, budget, permissions, and cleanup. Never commit AWS credentials or real secrets.

## 7-hour plan
1. **Hour 1:** EKS architecture, control plane, worker nodes, managed node groups.
2. **Hour 2:** Terraform workflow and safe EKS infrastructure planning.
3. **Hour 3:** IAM roles for service accounts (IRSA) and least privilege.
4. **Hour 4:** PV/PVC and persistent application data.
5. **Hour 5:** Auto-healing, readiness/liveness probes, and rollout strategies.
6. **Hour 6:** HPA metrics and External Secrets Operator (ESO) flow.
7. **Hour 7:** Troubleshooting, interview practice, documentation and cleanup.

## 1. EKS architecture

Amazon EKS is AWS's managed Kubernetes service. AWS operates the Kubernetes control plane; you are responsible for worker compute and the workloads/configuration you deploy, plus network, IAM and security choices.

Typical path:

```text
Terraform
  ├── VPC, subnets, routing and security groups
  ├── EKS cluster (managed control plane)
  ├── Managed node group / worker capacity
  └── IAM roles and policies
             |
          kubectl
             |
     Kubernetes Deployment
             |
        Pods on nodes
```

A managed node group automates provisioning and lifecycle management for EC2 worker nodes. It does not remove the need to plan capacity, patching, scaling, security, availability zones, or cost.

## 2. Terraform workflow for EKS

Recommended workflow:
1. Review AWS account, region, permissions, and budget.
2. Read the module/provider documentation and pin tested versions.
3. Keep Terraform state in a protected backend for team use.
4. Run formatting, validation, and a plan review.
5. Apply only with explicit approval in an authorized sandbox.
6. Verify the cluster and workloads.
7. Destroy lab resources and verify remaining billable resources.

Typical commands:

```bash
terraform fmt -recursive
terraform init
terraform validate
terraform plan -out=tfplan
terraform show tfplan
# Only after review and approval:
terraform apply tfplan
```

Do not run `apply` just to test a tutorial. A plan can still reveal expensive resources, and state may contain sensitive values. Do not commit `tfplan`, state files, credentials, or backend secrets.

For a real implementation, use a maintained EKS Terraform module or a reviewed in-house module. Verify module versions, supported Kubernetes versions, subnet tags, cluster endpoint access, node group configuration, IAM roles, and add-ons against current AWS documentation before provisioning. This lesson intentionally does not prescribe a full paid-cluster configuration.

## 3. Connect to an existing authorized EKS cluster

Prerequisites: AWS CLI, kubectl, and permission to access the intended cluster.

```bash
aws sts get-caller-identity
aws configure list
aws eks list-clusters --region YOUR_REGION
aws eks update-kubeconfig --region YOUR_REGION --name YOUR_CLUSTER
kubectl config current-context
kubectl get nodes -o wide
kubectl get namespaces
```

Replace placeholders with values from your approved environment. Verify the AWS account and Kubernetes context before making changes. If you have not created a cluster, do not expect these commands to work.

## 4. IRSA: IAM Roles for Service Accounts

IRSA lets a Kubernetes ServiceAccount assume a specific IAM role through EKS's OIDC identity provider. It supports least-privilege AWS access for a workload without putting long-lived AWS access keys inside a Pod.

Conceptual flow:
1. EKS has an OIDC issuer.
2. An IAM role trust policy trusts the intended OIDC provider and exact ServiceAccount subject/audience.
3. The role policy grants only the AWS actions and resources required.
4. A Kubernetes ServiceAccount is annotated with the role ARN.
5. The application uses the AWS SDK/credential chain to obtain temporary credentials.

Example ServiceAccount shape (replace the role ARN; this annotation alone does not configure IAM trust or permissions):

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-reader
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT_ID:role/APP_ROLE
```

Security checks:
- Trust policy must constrain the correct OIDC issuer and ServiceAccount subject.
- IAM policy should use least privilege.
- Do not place AWS access keys in images, manifests, ConfigMaps, or source control.
- Check whether your organization uses EKS Pod Identity instead; it is a distinct mechanism and should not be confused with IRSA.

## 5. Persistent storage: PV and PVC

- **PersistentVolume (PV):** cluster storage resource.
- **PersistentVolumeClaim (PVC):** a workload's request for storage.
- **StorageClass:** describes dynamic provisioning behavior.
- On EKS, the EBS CSI driver commonly provisions EBS-backed volumes. A suitable driver/add-on, IAM permissions, StorageClass and cluster setup are required.

Example PVC for a cluster with a compatible default StorageClass:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Apply and inspect only in a suitable lab:

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
kubectl describe pvc app-data
kubectl get storageclass
kubectl get pv
```

A PVC may remain `Pending` if no compatible StorageClass exists, the CSI driver is missing, or the requested access mode/capacity cannot be satisfied. EBS volumes are commonly zonal; scheduling and volume topology matter. Do not assume the PVC will bind in every cluster.

## 6. Auto-healing and rollout strategies

Kubernetes controllers reconcile actual state toward desired state:
- A Deployment/ReplicaSet replaces Pods that disappear.
- A node failure can cause Pods to be rescheduled if capacity and storage constraints permit.
- Readiness probes control whether Pods are considered ready for Service traffic.
- Liveness probes can trigger container restarts.
- Startup probes can protect slow-starting containers from premature liveness failures.

Inspect workloads:

```bash
kubectl get deployments,pods -o wide
kubectl describe deployment APP_NAME
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl rollout status deployment/APP_NAME
kubectl rollout history deployment/APP_NAME
kubectl rollout undo deployment/APP_NAME
```

Common Deployment strategies:
- **RollingUpdate:** gradually replaces old replicas; control with `maxSurge` and `maxUnavailable`.
- **Recreate:** terminates old Pods before creating new ones; can cause downtime.
- **Blue/green:** keeps two versions and switches traffic after verification; often implemented with Service selectors or a routing layer.
- **Canary:** gradually exposes a small share of traffic to a new version; usually needs a capable ingress/service mesh or traffic-management layer.

A Deployment rollback does not reverse database schema changes or external side effects. Plan backward-compatible migrations and observe the app during rollout.

## 7. HPA: Horizontal Pod Autoscaler

HPA adjusts the replica count based on observed metrics such as CPU utilization. It typically needs the Kubernetes Metrics API (often metrics-server) and resource requests for CPU utilization calculations.

Check prerequisites and status:

```bash
kubectl top nodes
kubectl top pods
kubectl get hpa
kubectl describe hpa APP_NAME
```

If metrics are unavailable, HPA may show unknown metrics and will not scale as expected. For a lab Deployment with resource requests set, an HPA can be created using:

```bash
kubectl autoscale deployment APP_NAME --cpu-percent=60 --min=2 --max=6
kubectl get hpa
```

Replace `APP_NAME` with a real Deployment. This example is for practice, not a universal production threshold. HPA controls Pod replicas, not the number of EC2 nodes; node autoscaling is a separate capacity mechanism.

## 8. External Secrets Operator (ESO)

ESO synchronizes secret values from an external secrets provider into Kubernetes Secret objects. A typical AWS Secrets Manager flow is:

1. Install and secure ESO using its official, version-matched documentation.
2. Give ESO's controller identity only the required AWS permissions, using an approved identity mechanism.
3. Configure a `SecretStore` or `ClusterSecretStore`.
4. Define an `ExternalSecret` that references a secret by name/key.
5. Verify synchronization status and access controls.

Illustrative structure only — provider fields, authentication configuration, API versions and CRDs depend on the installed ESO release and provider setup:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-config
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-store
    kind: SecretStore
  target:
    name: app-config
  data:
    - secretKey: API_TOKEN
      remoteRef:
        key: my-app/production
        property: API_TOKEN
```

Do not apply this example until the matching ESO CRDs, store, authentication, and provider version are configured. The example references a placeholder secret; it does not create one. Restrict who can read the resulting Kubernetes Secret and avoid printing secret values.

## Troubleshooting

| Symptom | Investigate |
|---|---|
| `kubectl` Unauthorized / forbidden | AWS identity, EKS access entry or aws-auth configuration for the cluster, Kubernetes RBAC and current context |
| No nodes | Managed node group status, subnet routing, IAM permissions, capacity and security groups |
| Pod Pending | Node capacity, taints/tolerations, resource requests, PVC status and events |
| PVC Pending | StorageClass, CSI driver, IAM permissions, access mode and topology |
| HPA shows unknown metrics | Metrics API/metrics-server, resource requests, HPA events |
| IRSA access denied | ServiceAccount annotation, OIDC provider, trust policy subject/audience, IAM policy and Pod identity |
| ESO not syncing | CRD/API version, store readiness, AWS permissions, secret name/key and controller events |
| Rollout stalled | Pod readiness, image pull, probes, events and application logs |

Useful commands:

```bash
kubectl describe pod POD_NAME
kubectl logs POD_NAME --all-containers=true
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe hpa HPA_NAME
kubectl describe pvc PVC_NAME
```

## Interview questions

1. **What does EKS manage?** AWS manages the Kubernetes control plane; you still manage workload design and associated compute, networking, IAM and operational responsibilities.
2. **What is a managed node group?** An EKS feature that provisions and manages the lifecycle of EC2 worker nodes.
3. **What is IRSA?** A way for a Kubernetes ServiceAccount to obtain scoped AWS permissions through an OIDC trust relationship and IAM role.
4. **PV vs PVC?** PV represents storage capacity; PVC is a workload's claim/request for storage.
5. **What does HPA scale?** The replica count of a workload based on metrics; node scaling is separate.
6. **How does Kubernetes self-heal?** Controllers reconcile desired state by replacing failed Pods and taking other corrective actions, subject to available capacity and constraints.
7. **What does ESO do?** It syncs data from an external secret provider into Kubernetes Secret resources according to configured stores and ExternalSecrets.
8. **Rolling update vs blue/green?** Rolling update gradually replaces replicas; blue/green maintains two versions and switches traffic after validation.

## Cost-aware cleanup and verification

If you created AWS resources, use the exact Terraform configuration and workspace that created them. Review the destroy plan before approval:

```bash
terraform plan -destroy
# Only after verifying account, workspace, resources and costs:
terraform destroy
```

Do not destroy shared or production resources. Confirm no EKS cluster, node group, load balancer, NAT gateway, EBS volume, snapshots, or other billable resources remain. Kubernetes deletion alone does not necessarily remove every AWS resource. If you used only a local cluster, clean up only the local resources you created.

## Day 30 checklist
- [ ] Explain EKS control plane vs worker nodes and managed node groups.
- [ ] Run Terraform fmt/init/validate/plan safely; understand the cost of apply.
- [ ] Explain IRSA and least-privilege trust/policy configuration.
- [ ] Create or inspect a PVC and diagnose its binding status.
- [ ] Explain self-healing, probes and rollout strategies.
- [ ] Inspect HPA metrics and understand its prerequisites.
- [ ] Explain ESO's store, ExternalSecret and Kubernetes Secret flow.
- [ ] Record actual lab outputs and errors; never claim unrun commands passed.
- [ ] Verify cleanup and any AWS charges/resources.

**Next:** Day 31 — Advanced Kubernetes & GitOps: GitOps workflows, Argo CD, Helm/Kustomize patterns, and the TLS-secured EKS application project described in the course.
