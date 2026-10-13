# Day 12 — IAM, Security & Governance

## Course Alignment

This follows **Session 12: IAM, Security & Governance** from the DevOps & Cloud Engineering course plan.

Topics:
- IAM users, groups, roles, and policies
- Least privilege, MFA, and credential safety
- Organizations and Service Control Policies (SCPs)
- AWS KMS and Secrets Manager
- Secrets-sync concepts: External Secrets Operator (ESO) and HashiCorp Vault
- Security services overview: GuardDuty and CloudTrail

## Learning Objectives

By the end of Day 12, I should be able to explain IAM identities and policies, reason about least privilege, MFA and credential safety, describe Organizations and SCPs, explain KMS and Secrets Manager, describe ESO and Vault secret workflows, and distinguish GuardDuty from CloudTrail.

## 1. IAM Building Blocks

AWS Identity and Access Management (IAM) controls who can authenticate and what actions they are authorized to perform.

- **User:** an identity for a person or workload when a long-lived IAM user is appropriate. Prefer federation or temporary credentials where supported instead of creating long-lived access keys.
- **Group:** a collection of IAM users to which permissions can be assigned. Groups contain users, not roles.
- **Role:** an identity that trusted principals can assume to receive temporary credentials. Roles are commonly used by AWS services, workloads, and federated users.
- **Policy:** a JSON document that defines permissions. Policies may be identity-based, resource-based, or another supported policy type.
- **Principal:** the identity making a request, such as a user, role, or AWS service.

A role's **trust policy** determines who can assume it; its **permissions policies** determine what it can do after assumption. These are different questions.

### Example: Read-only access to one S3 bucket

This is a learning example. Replace the placeholder bucket name before testing, and use a sandbox account.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "ListOneBucket",
          "Effect": "Allow",
          "Action": "s3:ListBucket",
          "Resource": "arn:aws:s3:::example-learning-bucket"
        },
        {
          "Sid": "ReadObjectsInOneBucket",
          "Effect": "Allow",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::example-learning-bucket/*"
        }
      ]
    }

Important: **s3:ListBucket** applies to the bucket ARN; **s3:GetObject** applies to object ARNs. An Allow does not override an applicable explicit Deny, and the final result depends on all relevant policies and controls.

## 2. How to Read an IAM Policy

Common policy elements:
- **Version:** policy-language version, commonly 2012-10-17.
- **Statement:** one or more permission statements.
- **Effect:** Allow or Deny.
- **Action / NotAction:** API actions affected by the statement.
- **Resource / NotResource:** resources affected by the statement.
- **Condition:** optional context-based restrictions, such as source IP or requested Region.
- **Principal:** appears in policies that specify who can access a resource or assume a role, such as trust policies.

### Authorization checklist

When troubleshooting AccessDenied, check:
1. Which principal is making the request?
2. Is it using the expected user or assumed role?
3. Which action and resource are involved?
4. Is there an applicable identity-based or resource-based policy?
5. Is there an explicit deny, permissions boundary, session policy, SCP, or other applicable control?
6. Are the resource ARN, condition values, account, and Region correct?

Do not respond to an access error by attaching AdministratorAccess as a shortcut. Identify the missing permission and grant only what is justified.

## 3. Least Privilege, MFA & Credential Safety

**Least privilege** means granting only the actions and resources needed for a defined task, for only as long as needed.

Practical controls:
- Use IAM roles and temporary credentials for workloads whenever possible.
- Enable MFA for human access, especially privileged access.
- Avoid root-user access keys; protect root access and use it only for tasks that require it.
- Do not share credentials between people or workloads.
- Never commit access keys, passwords, session tokens, private keys, or secret values to Git.
- Store secrets in an approved secret manager, not in source code, container images, or plain-text configuration.
- Rotate or revoke exposed credentials promptly and investigate possible use.
- Review unused identities, keys, permissions, and roles periodically.
- Prefer short-lived sessions and auditable access paths.

If credentials are accidentally committed, deleting the file or rewriting Git history is not enough. Treat the credential as exposed: revoke or rotate it, investigate usage, and then clean up repository history as appropriate.

## 4. AWS Organizations & Service Control Policies

**AWS Organizations** helps centrally manage multiple AWS accounts and apply governance.

- Organize accounts into organizational units (OUs).
- Use consolidated governance and account-management workflows.
- Apply **Service Control Policies (SCPs)** to define the maximum available permissions for principals in member accounts.
- An SCP does **not** grant permissions. IAM and other applicable authorization controls must still allow the requested action.
- SCPs generally do not restrict actions performed by the Organizations management account in the same way they restrict member accounts; verify current AWS documentation for exact behavior and exceptions.

### Governance exercise — design before deploying

Draft a policy plan for a sandbox organization:
- Which accounts are for development, testing, and production?
- Which actions should be prohibited at the organization level?
- Who can create IAM roles and change security settings?
- How will emergency access and policy changes be reviewed?
- How will you test that the policy does not block required operations?

Do not attach an SCP or change organization-wide controls in a shared or production organization as a beginner lab. Use an authorized sandbox and review the impact first.

## 5. AWS KMS & Secrets Manager

### AWS Key Management Service (KMS)

KMS helps create and control cryptographic keys used to protect data.

- Key policies and IAM policies both matter to access decisions.
- Use least privilege for key administration and key usage.
- Understand the difference between permission to administer a key and permission to use it for cryptographic operations.
- Monitor key usage and follow an approved rotation and lifecycle strategy.
- Encryption does not remove the need to control who can access plaintext data.

### AWS Secrets Manager

Secrets Manager stores and manages secrets such as database credentials and API tokens.

Good practices:
- Retrieve secrets at runtime using an authorized workload role.
- Avoid printing secret values in logs or command output.
- Restrict which principals can read, update, or delete each secret.
- Consider rotation where supported and operationally appropriate.
- Use KMS-backed encryption and monitor access.
- Keep secret identifiers/configuration separate from the secret values themselves.

Never put real credentials into this learning repository. Use placeholders in examples.

## 6. Secrets Synchronization Concepts: ESO & Vault

Applications running in Kubernetes or other platforms need a safe way to consume secrets without hardcoding them into manifests or images.

- **External Secrets Operator (ESO):** a Kubernetes operator that can read secrets from an external provider and synchronize selected values into Kubernetes Secret objects, based on configured resources and permissions.
- **HashiCorp Vault:** a secrets-management system that can centralize secret storage, access control, auditing, and dynamic-secret workflows depending on configuration and integrations.
- **AWS Secrets Manager:** an AWS-managed service for storing and managing secrets.

Conceptual flow:

    Workload identity / IAM role
              |
              v
    Approved secret provider
    (e.g. AWS Secrets Manager or Vault)
              |
              v
    ESO or application integration
              |
              v
    Workload consumes secret securely

Security notes:
- Grant the controller or workload access only to the specific secrets it needs.
- Protect Kubernetes Secret access; a Kubernetes Secret object is not automatically safe merely because it is called a Secret.
- Do not commit secret values or unencrypted secret manifests.
- Plan for rotation, synchronization failures, access revocation, and audit trails.

ESO and Vault are concepts to understand today; implementing them in a Kubernetes cluster belongs to a later lab in the course.

## 7. GuardDuty & CloudTrail

These services have different primary purposes:

- **AWS GuardDuty:** threat-detection service that analyzes supported data sources and findings to help identify potentially malicious or suspicious activity.
- **AWS CloudTrail:** records AWS account activity and API events, helping answer who performed an action, when it happened, and which API activity was recorded.

Example operational workflow:
1. A security finding appears in GuardDuty.
2. Use the finding details to scope the affected account, resource, and time window.
3. Review relevant CloudTrail events and other evidence.
4. Determine whether the activity was expected or suspicious.
5. Follow the incident-response process, including credential revocation or containment if warranted.
6. Document evidence, root cause, remediation, and prevention.

Neither service replaces correct IAM design, patching, network controls, or an incident-response plan.

## 8. Hands-On Lab — IAM Policy Reasoning in a Sandbox

Use an AWS account you are authorized to manage. Prefer a dedicated learning account. Avoid changing root access, organization-wide policies, or production roles during this exercise.

1. **Inspect:** open IAM and identify users, groups, roles, and policies that you are permitted to view.
2. **Identify the principal:** check which identity or assumed role your current session uses.
3. **Read a policy:** find its Effect, Action, Resource, and any Condition blocks.
4. **Use a harmless test:** create or use a dedicated sandbox role/user with narrowly scoped read-only access to a test resource, if your account permissions allow it.
5. **Validate both outcomes:** confirm the intended read operation is allowed and an unrelated operation remains denied. Do not test against production data.
6. **Inspect MFA and credentials:** review the account's permitted MFA and access-key settings without exposing any secret values.
7. **Explore governance:** read about Organizations and SCPs; do not attach an SCP unless this is an approved isolated sandbox and you have reviewed the impact.
8. **Explore security visibility:** inspect available CloudTrail event history and GuardDuty findings if enabled. Availability depends on account setup, Region, permissions, and service configuration.
9. **Document:** record the principal, intended action, resource scope, test result, and policy reasoning. Redact account-specific identifiers if publishing publicly.
10. **Clean up:** remove only disposable resources created for this lab and verify there are no unwanted billable resources.

If you do not have permission to create IAM resources, complete the policy-reading and authorization-troubleshooting exercises without changing account configuration.

## 9. Failure Engineering & Troubleshooting

Investigate these scenarios conceptually or in a controlled sandbox:

- **AccessDenied:** verify principal, action, resource ARN, conditions, explicit denies, permissions boundaries, session policies, and SCPs.
- **A workload cannot assume a role:** inspect the role trust policy, caller identity, and required sts:AssumeRole permissions.
- **A workload cannot read a secret:** check the secret's resource, caller role, IAM permissions, KMS permissions where applicable, and Region.
- **A secret is exposed in a repository:** revoke/rotate it first, investigate usage, remove the exposure, and add preventive scanning/process controls.
- **An unexpected API call appears:** correlate the CloudTrail event with the principal, source, time, and surrounding events; check related GuardDuty findings.
- **A policy change breaks a service:** compare the change with the failed action and resource, restore a known-good configuration through the approved process, and add a regression test.

For each scenario, record symptom, evidence, root cause, safe remediation, and prevention. Do not broaden permissions blindly.

## 10. DevOps Connection

IAM and governance underpin secure automation. CI/CD pipelines, EC2 instances, container workloads, and Kubernetes integrations should use narrowly scoped roles, temporary credentials, auditable access, and managed secrets. Security controls should be repeatable and reviewed alongside infrastructure changes.

## 11. Interview Questions

1. What is the difference between an IAM user and an IAM role?
2. What is the difference between a role trust policy and a permissions policy?
3. How do Action, Resource, Condition, Allow, and Deny work in an IAM policy?
4. What does least privilege mean in practice?
5. Why is MFA important, and how should workload credentials be handled?
6. Does an SCP grant permissions? Explain.
7. What is the difference between KMS and Secrets Manager?
8. What problem do ESO and Vault help solve?
9. What is the difference between GuardDuty and CloudTrail?
10. How would you investigate an AccessDenied error?
11. What should you do first if an AWS access key is exposed in a public repository?
12. How can CI/CD workloads access AWS without storing long-lived access keys?

## 12. DSA / CS Fundamentals

### Problem — Valid Anagram (Revision)

Given two strings, determine whether one is an anagram of the other.

- Typical approach: count character frequencies or compare frequency maps.
- Time complexity: **O(n)** expected for a frequency-map approach.
- Space complexity: **O(k)** for the distinct character set, where k is the number of distinct characters.

Explain why matching lengths alone is insufficient and how frequency counts establish correctness.

## 13. Day 12 Checkpoint

Before moving on, explain without notes:
- The difference between IAM users, groups, roles, and policies
- How a role trust policy differs from its permissions policy
- How you would troubleshoot AccessDenied
- Least privilege, MFA, and safe credential handling
- Why an SCP limits permissions but does not grant them
- The difference between KMS and Secrets Manager
- The purpose of ESO and Vault in secret workflows
- The difference between GuardDuty and CloudTrail

### Learning Report

Record:
- IAM policy reviewed and what it allows
- Principal and resource used in your test
- Allowed and denied actions observed (if tested)
- MFA/credential-safety controls reviewed
- One AccessDenied scenario analyzed
- Whether CloudTrail event history or GuardDuty findings were available
- Any sandbox resources created and cleaned up

Do not include access keys, passwords, tokens, secret values, or sensitive account information in the report.

## Next

**Day 13 — VPC & AWS Networking Deep Dive**

Topics: VPCs, public/private subnets, route tables, Internet Gateway and NAT, Security Groups versus NACLs, VPC peering, Transit Gateway, Route 53, and building a custom VPC end-to-end.
