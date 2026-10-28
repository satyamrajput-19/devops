# Day 31 — Advanced Kubernetes & GitOps

## Course scope
The course Session 31 covers advanced Kubernetes and GitOps, including a TLS-secured EKS application with Nginx Ingress and Cert-Manager deployed via Argo CD. This lesson focuses on Argo CD, Helm, Kustomize and the EKS/TLS workflow.

## Learning objectives
- Explain GitOps and desired-state reconciliation.
- Understand Argo CD's role in deploying Kubernetes resources from Git.
- Package apps with Helm charts and values.
- Use Kustomize bases and overlays for environment differences.
- Understand TLS delivery with Ingress and Cert-Manager.
- Troubleshoot sync, rendering, health and certificate issues.

> **Cost and security:** Start with local Minikube. EKS, load balancers, public IPs, NAT gateways and storage can incur charges. Do not create paid infrastructure without authorization, a budget and a reviewed cleanup plan. Never commit credentials, kubeconfigs, private keys or real secrets.

## 7-hour plan
1. Hour 1: GitOps principles and Argo CD architecture.
2. Hour 2: Install/inspect Argo CD on a local lab cluster.
3. Hour 3: Helm chart structure and values.
4. Hour 4: Kustomize bases and overlays.
5. Hour 5: Connect Git to an Argo CD Application and verify reconciliation.
6. Hour 6: EKS, Nginx Ingress, TLS and Cert-Manager architecture.
7. Hour 7: Troubleshooting, interview prep, notes and cleanup.

## 1. GitOps mental model

Git stores the desired configuration; a controller observes the repository and cluster and reconciles the cluster toward the desired state.

    Developer change
         |
    Pull request + review
         |
    Merge to Git (desired state)
         |
    Argo CD detects repository revision
         |
    Render Helm / Kustomize / YAML
         |
    Compare desired vs live state
         |
    Sync resources to Kubernetes
         |
    Health/status checks + drift detection

Argo CD can use manual or automated sync. A Git commit does not mean the cluster has already changed: repository refresh, sync policy, permissions, rendering and health all matter. GitOps does not replace access controls, reviews, tests, secret management or observability.

## 2. Check your local cluster

    kubectl config current-context
    kubectl get nodes
    minikube status

If you intend to use Minikube:

    minikube start --driver=docker
    kubectl get nodes -o wide

Verify that the current context is your disposable local cluster before installing components.

## 3. Install Argo CD locally (lab only)

Use the official, current Argo CD installation documentation for the release you choose. The common namespace setup is:

    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    kubectl get pods -n argocd
    kubectl rollout status deployment/argocd-server -n argocd

This pulls an upstream manifest. Review the source and release/version before applying it anywhere beyond a lab. For production, pin and review versions, apply security controls and use a documented upgrade path.

Port-forward the UI locally:

    kubectl port-forward svc/argocd-server -n argocd 8080:443

Then open https://localhost:8080. A local lab may show a self-signed certificate warning. Do not bypass TLS warnings on real production endpoints. Initial admin credentials and login details can vary by version/install method; follow official documentation and never paste credentials into GitHub or chat.

Inspect:

    kubectl get applications -n argocd
    kubectl get pods -n argocd
    kubectl get events -n argocd --sort-by=.metadata.creationTimestamp

## 4. Helm: package Kubernetes resources

A Helm chart contains templates and default values. Values configure an application without editing every template.

Typical structure:

    my-chart/
      Chart.yaml
      values.yaml
      templates/
        deployment.yaml
        service.yaml

Useful commands:

    helm version
    helm lint ./my-chart
    helm template my-release ./my-chart
    helm list -A

- helm lint checks chart structure.
- helm template renders manifests locally for inspection.
- An Argo CD Application can specify Helm value files.
- Do not store credentials in plain values.yaml committed to Git.

The chart schema and values depend on the application. Review rendered manifests before deployment.

## 5. Kustomize: base + overlays

Kustomize composes and patches YAML. A common pattern shares a base and applies environment-specific overlays.

    k8s/
      base/
        deployment.yaml
        service.yaml
        kustomization.yaml
      overlays/
        dev/
          kustomization.yaml
        prod/
          kustomization.yaml

Example base kustomization.yaml:

    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization
    resources:
      - deployment.yaml
      - service.yaml
    commonLabels:
      app.kubernetes.io/part-of: demo-app

Example dev overlay at overlays/dev/kustomization.yaml:

    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization
    namespace: demo-dev
    resources:
      - ../../base
    replicas:
      - name: web
        count: 1

Render before applying:

    kubectl kustomize k8s/overlays/dev

Use kubectl apply -k k8s/overlays/dev only when you intend to apply that environment to the current cluster. Ensure the namespace exists and the referenced Deployment is named web.

## 6. Argo CD Application pattern

An Argo CD Application specifies the repository/path/revision to watch and the destination cluster/namespace. Example for a Kustomize overlay; replace placeholders with a repository you control.

    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: demo-dev
      namespace: argocd
    spec:
      project: default
      source:
        repoURL: https://github.com/ORG/REPO.git
        targetRevision: main
        path: k8s/overlays/dev
      destination:
        server: https://kubernetes.default.svc
        namespace: demo-dev
      syncPolicy:
        automated:
          prune: false
          selfHeal: false
        syncOptions:
          - CreateNamespace=true

This is a teaching example. Replace ORG/REPO and the path; ensure the target cluster, RBAC, repository access and project policy are correct. With prune disabled, resources removed from Git are not automatically deleted. Enabling pruning requires deliberate review. With selfHeal disabled, out-of-band live changes may not be automatically reverted. Automated sync, pruning and self-healing are policy decisions, not defaults to enable blindly.

Inspect status:

    kubectl get applications -n argocd
    kubectl describe application demo-dev -n argocd

A successful Git fetch is not the same as a healthy app. Check sync status, health, events and the actual workload.

## 7. TLS-secured EKS project: architecture

The course project describes a TLS-secured EKS app using Nginx Ingress and Cert-Manager, deployed via Argo CD. Typical high-level flow:

    DNS hostname
       |
    AWS load balancer / ingress entry point
       |
    Nginx Ingress Controller
       |
    TLS certificate managed by Cert-Manager
       |
    Ingress routes to Kubernetes Service
       |
    Deployment / Pods

Prerequisites:
- An EKS cluster and worker capacity.
- A compatible Nginx Ingress Controller and load-balancer setup.
- DNS pointing to the correct external endpoint.
- Cert-Manager installed with matching CRDs.
- An Issuer or ClusterIssuer configured for the chosen validation method.
- An Ingress with matching hostname, TLS secret and backend Service.
- Argo CD repository access and a reviewed GitOps path.
- IAM/network permissions and a cleanup plan.

Do not apply Ingress or Cert-Manager manifests without matching controller/CRD versions and a real DNS/issuer plan. A local Minikube port-forward teaches local routing but does not prove public TLS issuance or production EKS behavior.

## Troubleshooting

| Symptom | What to check |
|---|---|
| Argo CD app OutOfSync | Git revision/path, rendered manifests, live drift and sync policy |
| Sync fails | Application events, permissions, invalid schema and admission errors |
| Helm render fails | Chart version, required values, helm lint and helm template |
| Kustomize build fails | Relative paths, file names, API versions and resource references |
| App Degraded / Progressing | Pod status, readiness probes, logs, events and Service endpoints |
| Private repository unavailable | Securely configured repository credentials, URL and permissions |
| Ingress has no address | Ingress controller, Service/load balancer, cloud permissions and subnet configuration |
| Certificate not ready | Issuer status, DNS, challenge events, ACME limits and hostname consistency |

Useful commands:

    kubectl get applications -n argocd
    kubectl describe application APP_NAME -n argocd
    kubectl get pods,svc,ingress -A
    kubectl get events --sort-by=.metadata.creationTimestamp
    kubectl describe certificate CERT_NAME -n NAMESPACE
    kubectl describe challenge -A

Some resource types are available only after their controller and CRDs have been installed.

## Interview questions

1. **What is GitOps?** A practice where desired infrastructure/application state is versioned in Git and a controller reconciles the environment toward it.
2. **What does Argo CD do?** It compares Git-defined desired state with live Kubernetes state and reports or applies sync according to policy.
3. **Helm vs Kustomize?** Helm packages parameterized templates as charts; Kustomize composes and patches YAML through bases and overlays.
4. **What does OutOfSync mean?** The live state differs from rendered desired state or resources need synchronization.
5. **What is Argo CD self-healing?** A policy that can reapply Git-defined state when live resources drift; configure and govern it carefully.
6. **What does Cert-Manager do?** It automates certificate lifecycle management through configured issuers and certificate resources; it does not replace DNS or ingress configuration.
7. **Why use a pull request for GitOps?** It supports review, testing, traceability and approvals before desired-state changes reach the deployment branch.
8. **Does a green Argo CD sync guarantee the app works?** No. Validate health, probes, logs, traffic, dependencies and application behavior.

## Cleanup

If you installed Argo CD on a disposable local cluster and no longer need it:

    kubectl delete namespace argocd

This deletes Argo CD resources in that namespace. Check for application resources deployed elsewhere and clean up only resources you created. If you provisioned AWS resources, review the Terraform destroy plan and confirm all billable resources are handled. Do not delete shared or production resources.

## Day 31 checklist
- [ ] Explain GitOps desired state and reconciliation.
- [ ] Inspect Argo CD pods and application status in a local lab.
- [ ] Render a Helm chart and understand values.
- [ ] Build a Kustomize base and environment overlay.
- [ ] Understand an Argo CD Application source and destination.
- [ ] Explain TLS, Nginx Ingress and Cert-Manager prerequisites.
- [ ] Diagnose OutOfSync, rendering failures and certificate issues.
- [ ] Document actual execution results, not assumed success.
- [ ] Clean up lab resources safely.

**Next:** Day 32 — HA, DR, AWS Well-Architected, FinOps, and the career capstone.
