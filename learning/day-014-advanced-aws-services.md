# Day 14 — Advanced AWS Services

**Course phase:** AWS for DevOps & Cloud Engineers  
**Topics:** ALB/NLB, Auto Scaling, multi-AZ high availability, S3 storage classes, and Amazon ECS.

> Learning goal: explain how traffic reaches healthy targets, how capacity adjusts to demand, how availability improves across Availability Zones, how S3 storage classes trade retrieval cost against access patterns, and how ECS runs containerized workloads.

## 1. Elastic Load Balancing: ALB vs NLB

A load balancer distributes incoming traffic across registered targets and checks target health so traffic can be directed to healthy targets.

| Feature | Application Load Balancer (ALB) | Network Load Balancer (NLB) |
|---|---|---|
| Main layer | Layer 7 — HTTP/HTTPS | Layer 4 — TCP/UDP/TLS |
| Routing | Host/path/header-based rules | Connection-level routing; no HTTP path routing |
| Common use | Web apps, APIs, microservices, HTTP routing | High-throughput/low-latency network services, static IP needs |
| Health checks | Target-group health checks | Target-group health checks |

Choose ALB when routing depends on HTTP request content. Choose NLB when transport-level performance, protocols, or static IP support are central requirements. Validate protocol and feature requirements before choosing.

### Request path

Client → DNS → Load Balancer → Healthy Target → Application

A listener accepts connections on a protocol/port and applies its rules or forwards traffic to a target group. A target group defines registered targets and health-check behavior.

## 2. Auto Scaling

Amazon EC2 Auto Scaling helps maintain desired capacity by launching or terminating instances according to configured policies and health checks.

- **Minimum capacity:** lowest permitted group size.
- **Desired capacity:** target number of instances to maintain.
- **Maximum capacity:** upper limit for scaling.
- **Launch template:** defines instance settings such as AMI, instance type, security groups, and user data.
- **Scaling policy:** changes desired capacity based on metrics, schedules, or step adjustments.
- **Health checks:** identify unhealthy instances so they can be replaced.

Common policy types include target tracking, step scaling, and scheduled scaling. Target tracking can adjust capacity to maintain a chosen metric, such as average CPU utilization.

### Practical design checklist
1. Place instances in an Auto Scaling group spanning at least two Availability Zones.
2. Use a launch template and an appropriate instance profile.
3. Register instances with the load balancer target group.
4. Configure application and load-balancer health checks.
5. Set sensible minimum, desired, and maximum capacity.
6. Choose a scaling metric tied to workload demand.
7. Test scale-out and scale-in, and inspect logs and events.

Auto Scaling is not the same as high availability by itself. Application state, data storage, health checks, capacity limits, and dependencies also matter.

## 3. High availability across Availability Zones

High availability (HA) aims to reduce service interruption when a component or an Availability Zone fails. Multi-AZ architecture places redundant resources in separate Availability Zones within a Region.

Example architecture:

    Users
      |
    Route 53 / DNS
      |
    ALB across multiple AZs
      |                       |
    Target group           Target group
      |                       |
    EC2 Auto Scaling group across AZs
      |                       |
    Application instances in private subnets
      |
    Durable/shared data service designed separately for HA

Design principles:
- Use more than one AZ for the load balancer and application capacity.
- Keep application instances replaceable; avoid relying on local instance storage for durable user data.
- Use health checks that reflect real application readiness.
- Make deployments and startup behavior repeatable.
- Monitor availability, latency, errors, capacity, and scaling activity.
- Test failure scenarios and document recovery steps.
- Review dependencies separately; a single-AZ database can remain a single point of failure.

**Multi-AZ is not the same as multi-Region disaster recovery.** Multi-Region designs address a broader regional failure and require explicit data replication, traffic failover, recovery objectives, and testing.

## 4. Amazon S3 fundamentals and storage classes

Amazon S3 is object storage. Data is stored as objects in buckets, with keys identifying objects. S3 is commonly used for static assets, logs, backups, artifacts, data lakes, and application files.

Key concepts:
- **Bucket:** container for objects, created in an AWS Region.
- **Object/key:** the stored data and its key within a bucket.
- **Versioning:** preserves multiple versions and can help recover from accidental overwrites/deletions.
- **Lifecycle rules:** transition or expire objects based on age or other filters.
- **Encryption and access control:** use appropriate encryption, Block Public Access, IAM/bucket policies, and least privilege.

### Storage class selection

| Storage class | Suitable access pattern | Key consideration |
|---|---|---|
| S3 Standard | Frequently accessed data | General-purpose, low-latency access |
| S3 Intelligent-Tiering | Access patterns unknown or changing | Automatically moves eligible objects between access tiers; monitoring/automation charges may apply |
| S3 Standard-IA | Infrequently accessed data needing rapid access | Retrieval charges and minimum storage duration apply |
| S3 One Zone-IA | Infrequent access where re-creatable data can tolerate one-AZ storage | Not suitable as the only copy of critical data |
| S3 Glacier Instant Retrieval | Archive data needing millisecond retrieval | Retrieval and minimum-duration charges apply |
| S3 Glacier Flexible Retrieval | Archive data with flexible retrieval times | Retrieval may take minutes to hours depending on option |
| S3 Glacier Deep Archive | Very rarely accessed long-term archives | Very low-cost archival use cases; retrieval can take many hours |

Before selecting a class, compare storage price, retrieval charges, minimum storage duration, retrieval latency, availability needs, and lifecycle behavior. Check current AWS documentation and regional pricing because features and prices can change.

### Safe S3 lab
1. Create a uniquely named test bucket in the intended Region.
2. Keep Block Public Access enabled.
3. Upload a harmless sample file.
4. Enable versioning and upload a changed version; inspect versions.
5. Create a lifecycle rule for a test prefix only, if appropriate.
6. Review encryption, bucket policy, and IAM permissions.
7. Delete test objects and the bucket when finished, if no longer needed.

Never upload credentials, private data, or production files to a learning bucket.

## 5. Amazon ECS: running containers

Amazon Elastic Container Service (ECS) is a managed container orchestration service. It schedules and operates containers using AWS infrastructure.

- **Cluster:** logical grouping of capacity and workloads.
- **Task definition:** blueprint for one or more containers, including image, CPU/memory, ports, environment, logging, and IAM roles.
- **Task:** running instance of a task definition.
- **Service:** maintains the desired number of tasks and supports deployments and integration with load balancing.
- **Container image:** packaged application and dependencies, commonly stored in Amazon ECR.
- **Fargate:** AWS manages the underlying servers.
- **ECS on EC2:** you manage the EC2 capacity layer.

### Typical ECS deployment flow

Application code → build and test container image → push image to Amazon ECR → register ECS task definition → create/update ECS service → run tasks on Fargate or EC2 → configure ALB health checks if needed → inspect CloudWatch logs and metrics.

Operational checks:
- Use task and execution roles for different permissions; grant least privilege.
- Do not bake secrets into the image. Use suitable secret management and task permissions.
- Configure logs, health checks, CPU/memory, networking, and deployment settings.
- Pin images to an immutable tag or digest for reproducibility.
- For production, consider private subnets, controlled egress, TLS, and an ALB when HTTP routing is needed.

## 6. Hands-on lab plan

Use a sandbox account and check current charges before creating resources.

### Lab A — Load balancer and Auto Scaling design
- Draw a two-AZ architecture with public ALB subnets and private application subnets.
- Define an Auto Scaling group with a launch template and at least two AZs.
- Attach the group to an ALB target group.
- Configure health checks and a scaling policy.
- If implementing, use a minimal test application and verify healthy targets before testing traffic.

### Lab B — S3 storage and lifecycle
- Create a private test bucket, upload sample data, enable versioning, and inspect object versions.
- Describe which class fits frequent access, occasional access, and long-term archive.
- Create a narrowly scoped lifecycle rule only after reviewing transition/expiration consequences.

### Lab C — ECS service design
- Containerize a small web application.
- Push its image to ECR.
- Create a task definition and an ECS service using Fargate or an existing EC2 capacity setup.
- Configure logs, task roles, health checks, and (if needed) an ALB.
- Verify the service, then remove test resources.

**Cost warning:** ALBs/NLBs, NAT Gateways, EC2, Fargate tasks, data transfer, and other resources may incur charges. Monitor costs, delete test resources after testing, and confirm billable components are gone.

## 7. Troubleshooting checklist

**ALB/NLB**
- Is the listener configured for the expected protocol and port?
- Are targets registered and healthy?
- Do security groups, NACLs, and routes allow the required traffic?
- Does the application listen on the target port and return a successful health-check response?
- Check target health reason codes and load-balancer metrics.

**Auto Scaling**
- Check group activities, desired/min/max capacity, launch-template version, quotas, subnet IP capacity, and instance launch errors.
- Confirm scaling policy metrics and alarms are configured correctly.
- Verify health checks and instance bootstrap logs.

**S3**
- Check bucket/key spelling, IAM identity policy, bucket policy, encryption/KMS permissions, and Block Public Access settings.
- Distinguish AccessDenied from a missing object.
- Check versioning and lifecycle rules before deleting or transitioning data.

**ECS**
- Check service events, task stopped reason, image URI and ECR permissions, task/execution roles, CPU/memory, subnet routes, security groups, logs, and container health checks.
- If using an ALB, verify target-group port and health-check path.

## 8. Interview questions

1. When would you choose ALB instead of NLB?
2. What is the difference between a listener and a target group?
3. What do minimum, desired, and maximum capacity mean in an Auto Scaling group?
4. How does a load balancer help an Auto Scaling group replace unhealthy capacity?
5. Why should application instances be spread across Availability Zones?
6. What can still be a single point of failure in a multi-AZ web architecture?
7. What is the difference between S3 versioning and lifecycle management?
8. How do S3 Standard-IA and Glacier classes differ in access patterns and charges?
9. What is the difference between an ECS task definition, task, and service?
10. When might you choose Fargate instead of ECS on EC2?
11. How would you investigate ECS tasks that repeatedly stop?
12. How would you control cloud costs while running this lab?

## 9. DSA + daily checkpoint

**DSA practice — Binary Search**
- Given a sorted array and a target, return its index or -1.
- Target complexity: **O(log n)** time and **O(1)** extra space with an iterative implementation.
- Explain why binary search requires sorted input.

### Day 14 completion checklist
- [ ] Explain ALB vs NLB and listener/target-group behavior.
- [ ] Explain Auto Scaling minimum/desired/maximum capacity and scaling policies.
- [ ] Draw a multi-AZ web architecture and identify possible single points of failure.
- [ ] Choose an S3 storage class for three different access patterns.
- [ ] Explain ECS cluster, task definition, task, service, Fargate, and EC2 capacity.
- [ ] Complete at least one lab or document the design and troubleshooting steps.
- [ ] Add observations, screenshots with secrets/IPs redacted, and lessons learned.

### Learning report template
- **What I learned:**
- **What I built or designed:**
- **What failed and how I diagnosed it:**
- **One security/cost decision I made:**
- **One question I can now answer:**
- **What I still need to practise:**

**Next session:** Day 15 — Databases on AWS: RDS Multi-AZ, read replicas, backups, DynamoDB, ElastiCache, and SQL vs NoSQL.
