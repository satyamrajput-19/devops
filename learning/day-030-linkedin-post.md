🚀 **Day 30 — Kubernetes on AWS (EKS) & App Operations**

Today I studied how Kubernetes workloads operate on AWS EKS and the operational building blocks behind secure, resilient applications.

🔹 EKS architecture and managed node groups  
🔹 Terraform workflow for infrastructure provisioning  
🔹 IRSA and least-privilege AWS access for workloads  
🔹 PersistentVolumes (PV) and PersistentVolumeClaims (PVC)  
🔹 Kubernetes self-healing and rollout strategies  
🔹 Horizontal Pod Autoscaler (HPA) and metrics prerequisites  
🔹 External Secrets Operator (ESO) and external secret synchronization  
🔹 Troubleshooting and cost-aware cleanup

**Key takeaway:** EKS does not remove operational responsibility. IAM, networking, storage drivers, workload health, metrics, and cost controls all matter. HPA scales Pod replicas, while node scaling is a separate capacity concern. Secrets should never be committed to Git.

I documented the commands, example manifests, troubleshooting checks, and interview questions in my learning log. I’ll distinguish conceptual exercises from any AWS resources I actually provision.

📂 GitHub lesson: https://github.com/satyamrajput-19/devops/blob/main/learning/day-030-eks-app-operations.md

#AWS #EKS #Kubernetes #Terraform #DevOps #CloudEngineering #LearningInPublic
