# Day 10 — AWS Global Infrastructure & EC2

## Course Alignment

This follows **Session 10: AWS Global Infrastructure & EC2** from the DevOps & Cloud Engineering course plan.

Source topics:
- Global AWS structure: Regions, Availability Zones (AZs), and edge locations
- The Shared Responsibility Model
- EC2 components, AMIs, instance types, and pricing
- Launch and connect to your first EC2 instance

## Learning Objectives

By the end of Day 10, I should be able to:
- Explain the difference between an AWS Region, Availability Zone, and edge location.
- Describe the AWS Shared Responsibility Model.
- Explain what EC2, an AMI, and an instance type are.
- Compare the main factors that affect EC2 pricing.
- Plan and launch a small EC2 lab instance, then connect to it safely.
- Clean up lab resources to avoid unexpected charges.

## 1. AWS Global Infrastructure

### Regions
An AWS Region is a separate geographic area where AWS operates infrastructure. Regions are designed to be isolated from one another.

Choose a Region based on:
- Proximity to users and latency
- Service availability
- Data residency and compliance requirements
- Resilience requirements
- Cost

### Availability Zones (AZs)
An Availability Zone is an isolated location within a Region, made up of one or more discrete data centers with independent power, networking, and connectivity.

For higher availability, production designs often distribute workloads across multiple AZs rather than relying on one AZ.

### Edge Locations
Edge locations help deliver content and reduce latency for services such as Amazon CloudFront. They are part of AWS's global delivery infrastructure and are not the same thing as a Region or an AZ.

### Remember
- **Region** = geographic area
- **Availability Zone** = isolated location within a Region
- **Edge location** = point used by edge services to deliver content closer to users

## 2. AWS Shared Responsibility Model

AWS security responsibilities are shared between AWS and the customer.

### AWS is responsible for security *of* the cloud
Examples include:
- Physical data centers
- Physical hardware and underlying infrastructure
- The foundational infrastructure that runs AWS services

### The customer is responsible for security *in* the cloud
The exact split depends on the service. For an EC2 instance, customer responsibilities commonly include:
- Guest operating-system updates and hardening
- Identity and access management
- Network security configuration
- Application security
- Data protection and encryption choices
- Protecting credentials and secrets

**DevOps takeaway:** creating an EC2 instance does not automatically make its OS, application, credentials, or network exposure secure.

## 3. Amazon EC2 Basics

Amazon Elastic Compute Cloud (EC2) provides resizable virtual compute capacity.

Core components:
- **Instance:** the virtual server you run.
- **AMI (Amazon Machine Image):** a template containing information needed to launch an instance, including an OS image and configuration.
- **Instance type:** defines the compute, memory, storage, and networking capacity available to an instance.
- **Key pair:** used by supported connection methods to authenticate; protect the private key.
- **Security group:** virtual firewall controlling allowed inbound and outbound traffic.
- **EBS volume:** block storage that can be attached to an instance.
- **Public/private IP:** network addressing used to reach the instance, depending on its network setup.
- **User data:** optional launch-time instructions commonly used for initial configuration.

### Instance type selection
Choose a type based on workload requirements:
- General purpose: balanced compute, memory, and networking
- Compute optimized: compute-intensive workloads
- Memory optimized: memory-intensive workloads
- Storage optimized: workloads needing high local storage performance
- Accelerated computing: workloads that benefit from specialized hardware

Always check current regional availability and the exact instance specifications before choosing.

## 4. EC2 Pricing Fundamentals

Potential cost factors include:
- Instance type and running duration
- Operating-system or software licensing
- EBS volumes and snapshots
- Data transfer
- Elastic IP/public IPv4 usage and related networking features
- Additional services used by the workload

Common purchasing models include On-Demand, Savings Plans, and Reserved Instances; Spot Instances use spare AWS capacity and can be interrupted. These models have different terms and suitability.

For a first lab, prioritize a small eligible instance and review the current price in the AWS console before launching. **Free Tier eligibility and limits can vary by account, date, and service. Do not assume a resource is free.**

## 5. Hands-On Lab — Launch a Small EC2 Instance

### Before you start
- Sign in to the AWS Console.
- Select a Region and note it down.
- Check the current pricing and any account Free Tier eligibility.
- Use an account with appropriate permissions.
- Never paste AWS access keys, passwords, private keys, or other secrets into GitHub.

### Launch workflow
1. Open the **EC2** console and choose **Launch instance**.
2. Enter a clear name, such as `day10-ec2-lab`.
3. Choose a suitable Amazon Machine Image (AMI), such as an eligible Amazon Linux image.
4. Select a small instance type suitable for a learning lab and verify its price/eligibility.
5. Create or select a key pair if required by the chosen connection method. Download the private key securely and do not commit it to a repository.
6. Configure networking. Prefer a restricted inbound rule rather than opening management access to everyone.
7. Review storage and any additional settings.
8. Review the full configuration and launch the instance.
9. Wait for the instance to reach the running state and pass its status checks.
10. Record the Region, instance ID, AMI, instance type, and security-group rules in your private lab notes. Do not record secrets.

### Connecting
Use the connection method supported by the AMI and your configuration, such as EC2 Instance Connect, Session Manager (when configured), or SSH.

For SSH, a typical command looks like this; replace the placeholders with your actual values:

```bash
chmod 400 path/to/key.pem
ssh -i path/to/key.pem ec2-user@YOUR_INSTANCE_PUBLIC_IP
```

The default username depends on the AMI. Verify the correct username for the image you selected. SSH generally requires a suitable route and an inbound security-group rule limited to your trusted source IP.

Do not expose SSH (port 22) or RDP (port 3389) to `0.0.0.0/0` for convenience. Use a restricted source range or a managed access method where possible.

## 6. Verification and Troubleshooting

Check:
- Is the instance in the intended Region?
- Is its state `running`?
- Have instance status checks passed?
- Is the selected AMI compatible with the connection method?
- Is the username correct for that AMI?
- Does the instance have a reachable network path?
- Does the security group allow the required port from your source IP?
- Are the route table and subnet configuration appropriate?
- Is the key file protected and is the correct key being used?

Useful console areas:
- EC2 → Instances
- Instance details and status checks
- Security → Security groups
- Networking → subnet and IP details

## 7. Cleanup — Prevent Unexpected Charges

After recording what you learned:
1. Terminate the lab instance if you no longer need it.
2. Check for associated EBS volumes, snapshots, Elastic IPs, or other resources that may continue to incur charges.
3. Remove only lab resources you created and no longer need.
4. Review AWS Billing and Cost Management for current usage.

Stopping an instance does not necessarily stop every related charge. Confirm the state of associated resources.

## 8. Hands-On Tasks

### Lab 1 — Global Infrastructure
Write a short explanation of Region, Availability Zone, and edge location, with one example use case for each.

### Lab 2 — Shared Responsibility
Create a two-column table listing AWS responsibilities and customer responsibilities for an EC2 workload.

### Lab 3 — EC2 Design
Compare two available instance types for a hypothetical lightweight web server. Record vCPU, memory, networking considerations, regional availability, and estimated price from the console.

### Lab 4 — Launch and Connect
Launch a small lab instance, verify its status checks, and connect using a supported secure method. Capture screenshots only if they contain no credentials, private keys, account-sensitive information, or public details you do not want to disclose.

### Lab 5 — Cleanup
Terminate the instance when finished and check for related billable resources. Record what you cleaned up.

## 9. Failure Engineering

Test and explain these scenarios without weakening security:
- Wrong Region selected
- Instance still pending or status checks failing
- Incorrect SSH username
- Wrong private key or key permissions
- Inbound security-group rule missing or too broad
- No public IP or no valid route to the instance
- Instance type unavailable in the selected AZ
- Instance stopped but a storage resource remains

For each scenario, document:
1. The symptom
2. The evidence you checked
3. The root cause
4. The safe fix
5. How you would prevent recurrence

## 10. DevOps Connection

EC2 provides compute for workloads that DevOps teams provision, configure, patch, monitor, secure, and automate. Understanding Regions, AZs, IAM, networking, and cost is foundational before moving to production infrastructure and Infrastructure as Code.

## 11. Interview Questions

1. What is the difference between an AWS Region and an Availability Zone?
2. What are edge locations used for?
3. Explain the AWS Shared Responsibility Model.
4. What is an AMI?
5. How does an EC2 instance type affect workload performance?
6. What factors can contribute to EC2-related costs?
7. Why should SSH access be restricted to trusted source IPs?
8. Why might an EC2 instance be running but still unreachable?
9. Why is stopping an instance not always enough to eliminate charges?
10. How would you design for availability across multiple AZs?

## 12. DSA / CS Fundamentals

### Problem — Two Sum (Revision)
Given an array of integers and a target, return the indices of two values whose sum equals the target.

Example:
- Input: `nums = [2, 7, 11, 15]`, `target = 9`
- Output: `[0, 1]`

Practice a hash-map solution:
- Target time: **O(n)**
- Extra space: **O(n)**

Explain why checking previously seen values avoids a nested-loop search.

## 13. Day 10 Checkpoint

Before moving on, explain without notes:
- Region vs Availability Zone vs edge location
- Shared Responsibility Model
- EC2, AMI, instance type, security group, and EBS
- Main EC2 cost factors
- How you securely connect to an instance
- How you verify and clean up a lab

### Learning Report
Record:
- The Region and AZ concept you understood best
- The AMI and instance type you explored
- Which connection method you tested
- One security setting you reviewed
- One troubleshooting issue you investigated
- What resources you cleaned up

## Next

**Day 11 — Instance Configuration & Connectivity**

Topics: hardening and configuring EC2 for production; EBS types and snapshots; placement groups and the metadata service; Golden AMIs and AWS Systems Manager (SSM).
