# Day 11 — Instance Configuration & Connectivity

## Course Alignment

This follows **Session 11: Instance Configuration & Connectivity** from the DevOps & Cloud Engineering course plan.

Topics:
- Harden and configure EC2 for production
- EBS types, snapshots, placement groups, and the metadata service
- Golden AMIs and AWS Systems Manager (SSM)
- Build and test a secure baseline EC2 setup

## Learning Objectives

By the end of Day 11, I should be able to explain EC2 hardening, compare EBS volumes and snapshots, describe placement groups and instance metadata, explain Golden AMIs and Systems Manager, and test a secure baseline.

## 1. EC2 Hardening & Configuration

A secure baseline reduces unnecessary exposure and makes configuration repeatable.

- **Least privilege:** use an appropriate IAM role and avoid broad permissions.
- **Network exposure:** allow only required inbound traffic; restrict SSH/RDP to trusted sources or use Session Manager when configured.
- **Operating system:** apply security updates, remove unnecessary services, and use supported software.
- **Credentials:** protect private keys; never put credentials in source control, user data, logs, or screenshots.
- **Storage:** consider EBS encryption and control access to snapshots.
- **Monitoring:** use available logs, status checks, and Systems Manager capabilities.
- **Repeatability:** document the baseline and automate routine configuration where practical.

Verify the effect of each change and keep a safe recovery path. Do not practice on production instances.

## 2. Amazon EBS: Volumes & Snapshots

Amazon Elastic Block Store (EBS) provides persistent block storage for EC2.

| Type | General use |
|---|---|
| General Purpose SSD (gp3, gp2) | Boot volumes and general-purpose workloads |
| Provisioned IOPS SSD (io2, io1) | I/O-intensive workloads requiring provisioned performance |
| Throughput Optimized HDD (st1) | Throughput-oriented workloads; not a boot volume |
| Cold HDD (sc1) | Less frequently accessed throughput-oriented data; not a boot volume |

Check current AWS documentation, regional availability, performance limits, and pricing before choosing a volume.

- **Volume:** block storage attached to an EC2 instance in the same Availability Zone.
- **Snapshot:** point-in-time backup of an EBS volume, stored and managed by AWS.
- **Restore:** a snapshot can be used to create a new volume.
- **Encryption:** EBS encryption helps protect data at rest and supports encrypted snapshots and volumes.

A snapshot alone is not a complete disaster-recovery strategy. Consider recovery objectives, retention, access controls, and whether copies are needed in another Region or account.

### Lab — Inspect EBS safely

1. Open EC2 → Instances and select your lab instance.
2. Find its block-device mapping and open the associated EBS volume.
3. Record volume type, size, Availability Zone, encryption status, and attachment state.
4. Create a snapshot only if you understand its potential cost and have permission.
5. If practicing restore, create a separate test volume and verify it before deleting anything.
6. Clean up test volumes and snapshots that you no longer need.

Never delete a volume or snapshot until you have confirmed it is not needed.

## 3. EC2 Placement Groups

Placement groups influence how instances are placed on underlying hardware.

- **Cluster:** places instances close together for low-latency, high-throughput communication.
- **Spread:** places a small number of critical instances on distinct underlying hardware to reduce correlated failure risk.
- **Partition:** separates groups of instances into partitions, useful for large distributed workloads.

Placement groups do not replace application-level high availability, backups, or multi-AZ design. Check restrictions and supported instance types before using one.

## 4. EC2 Instance Metadata Service (IMDS)

The Instance Metadata Service provides information about an EC2 instance. It can also provide temporary credentials when an instance profile is configured.

Security points:
- Prefer **IMDSv2**, which uses a session-oriented token flow.
- Require IMDSv2 where compatible with your workloads.
- Restrict access to metadata from applications and processes that do not need it.
- Never log or expose temporary credentials.
- Treat server-side request forgery (SSRF) as a potential path to metadata access in vulnerable applications.

Example commands to run only from your own authorized test instance:

    TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
    curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id

Do not publish token values or credential responses.

## 5. Golden AMIs

A **Golden AMI** is a standardized machine image prepared with an approved operating-system baseline, required agents, configuration, and security updates.

A controlled workflow:
1. Start from a trusted base AMI.
2. Apply approved updates and configuration.
3. Install only required packages and agents.
4. Run security and functional checks.
5. Remove temporary credentials, secrets, and build artifacts.
6. Create and version the image.
7. Test instances launched from it.
8. Patch, replace, and retire older images through a documented lifecycle.

Golden AMIs improve consistency but can become stale. Define an owner, patch cadence, versioning scheme, and retirement process.

## 6. AWS Systems Manager (SSM)

Systems Manager helps operate and manage AWS resources. Capabilities depend on the agent, IAM permissions, network connectivity, and configuration.

Relevant capabilities:
- **Session Manager:** connect to supported instances without opening inbound SSH/RDP ports, when prerequisites are configured.
- **Run Command:** execute approved commands remotely on managed nodes.
- **Patch Manager:** help organize and apply operating-system patches.
- **Inventory and State Manager:** collect software/configuration information and maintain desired state through associations.

### Session Manager prerequisites

- The instance is configured as a Systems Manager managed node.
- SSM Agent is installed and running where required.
- The instance has an appropriate IAM instance profile and permissions.
- The instance can reach required Systems Manager endpoints.
- The operator has permission to start a session.

Do not assume SSM works automatically on every instance. Verify managed-node status and prerequisites first.

## 7. Hands-On Secure Baseline Lab

Use a disposable learning instance and follow your account's policies.

1. **Inspect:** record OS/AMI, instance type, security groups, IAM role, EBS configuration, and metadata options.
2. **Restrict access:** remove unnecessary inbound rules; keep management access limited to a trusted source or use a configured managed access method.
3. **Patch:** use a supported, documented method to check and apply OS updates.
4. **Storage:** inspect EBS type, encryption, and snapshot settings.
5. **Metadata:** inspect the instance metadata options and confirm whether IMDSv2 is required.
6. **SSM:** check whether the instance appears as a managed node; investigate agent, role, and connectivity prerequisites if it does not.
7. **Document:** record the baseline and verification evidence without secrets.
8. **Clean up:** remove only disposable resources you created and no longer need; check for billable volumes and snapshots.

## 8. Failure Engineering & Troubleshooting

Investigate these cases in a controlled lab:

- Session Manager cannot connect: check managed-node status, SSM Agent, IAM role, permissions, and endpoint connectivity.
- SSH is unreachable: check the source-IP rule, route, public/private addressing, key, and username.
- Snapshot restore does not behave as expected: check snapshot state, volume configuration, filesystem, and attachment/mount steps.
- A workload cannot access metadata: verify IMDS configuration and the application's supported token flow.
- A new instance differs from the intended baseline: compare AMI version, launch settings, user data, and configuration steps.
- A volume or snapshot remains after instance termination: inspect storage resources separately; do not assume all resources were removed.

For each issue, document the symptom, evidence, root cause, safe fix, and prevention step.

## 9. DevOps Connection

Secure instance configuration bridges provisioning and reliable operations. Repeatable baselines, managed access, patching, image versioning, storage recovery, and auditable changes are core practices for operating cloud infrastructure safely.

## 10. Interview Questions

1. What is EC2 hardening, and what belongs in a secure baseline?
2. What is the difference between an EBS volume and an EBS snapshot?
3. When would you choose gp3 versus a provisioned-IOPS SSD?
4. What are Cluster, Spread, and Partition placement groups?
5. What is EC2 instance metadata, and why is IMDSv2 important?
6. What is a Golden AMI, and how should it be maintained?
7. What is AWS Systems Manager Session Manager?
8. What prerequisites must be satisfied for Session Manager to work?
9. Why should SSH/RDP access be restricted?
10. Why are snapshots and Golden AMIs not substitutes for a tested recovery process?

## 11. DSA / CS Fundamentals

### Problem — Binary Search (Revision)

Given a sorted array and a target value, return the target's index or indicate that it is absent.

- Time complexity: **O(log n)**
- Extra space for an iterative solution: **O(1)**

Explain why each comparison lets you discard approximately half of the remaining search space. Binary search requires sorted input.

## 12. Day 11 Checkpoint

Before moving on, explain without notes:
- Three practical EC2 hardening controls
- The difference between EBS volumes and snapshots
- Use cases for the three placement-group strategies
- Why IMDSv2 matters
- How Golden AMIs support repeatable infrastructure
- The prerequisites and benefits of Session Manager

### Learning Report

Record:
- Security baseline settings inspected
- EBS volume type and encryption status
- Whether a snapshot was created and how you would restore it
- Instance metadata options checked
- Whether the instance was managed by Systems Manager
- One issue investigated and the evidence supporting your conclusion
- Lab resources cleaned up

## Next

**Day 12 — IAM, Security & Governance**

Topics: IAM users, groups, roles, and policies; least privilege, MFA, and credential safety; Organizations and SCPs; KMS and Secrets Manager; External Secrets Operator and Vault concepts; GuardDuty and CloudTrail.
