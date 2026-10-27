# Day 13 — VPC & AWS Networking Deep Dive

> DevOps & Cloud Engineering learning log · Session 13

## Learning objectives

By the end of this session, I should be able to explain how an AWS Virtual Private Cloud (VPC) isolates networking, design public and private subnets, trace traffic through route tables and gateways, compare Security Groups with Network ACLs, and describe VPC peering, Transit Gateway, and Route 53.

## 1. VPC fundamentals

A **VPC** is a logically isolated network in AWS where I define IP address ranges, subnets, routing, and network controls.

Key terms:
- **CIDR block:** the IP address range assigned to a VPC or subnet, for example `10.0.0.0/16`.
- **Subnet:** a range of VPC IP addresses associated with one Availability Zone.
- **Route table:** rules that determine where network traffic is sent.
- **Internet Gateway (IGW):** enables internet connectivity for resources with suitable routes and public addressing.
- **NAT Gateway:** lets resources in a private subnet initiate outbound IPv4 internet connections without accepting unsolicited inbound connections through that NAT path.
- **Security Group:** stateful virtual firewall attached to supported network interfaces/resources.
- **Network ACL (NACL):** stateless subnet-level traffic filter.

A subnet is not public merely because it has a public-looking CIDR. Its route table needs a route to an Internet Gateway, and a workload also needs appropriate addressing and security rules to be reachable from the internet.

## 2. Public and private subnets

A typical design separates internet-facing components from internal workloads.

Example address plan (illustrative only):

| Network | Example CIDR | Purpose |
|---|---|---|
| VPC | `10.0.0.0/16` | Private address space for this environment |
| Public subnet A | `10.0.1.0/24` | Internet-facing load balancer or NAT Gateway |
| Public subnet B | `10.0.2.0/24` | Multi-AZ public components |
| Private app subnet A | `10.0.11.0/24` | Application workloads |
| Private app subnet B | `10.0.12.0/24` | Application workloads in another AZ |
| Private data subnet A | `10.0.21.0/24` | Internal database tier |
| Private data subnet B | `10.0.22.0/24` | Internal database tier in another AZ |

This is a planning example, not a requirement to create all these resources in a personal account. Ensure CIDR ranges do not overlap with networks that may later be connected.

## 3. Route tables and gateways

Every subnet is associated with a route table. The most-specific matching route is selected for a destination.

Typical routes:
- Local VPC traffic: the VPC's local route is created automatically.
- Public subnet: `0.0.0.0/0 → Internet Gateway`.
- Private IPv4 subnet requiring outbound internet access: `0.0.0.0/0 → NAT Gateway` (with the NAT Gateway placed in a public subnet).
- A private subnet with no internet route can still communicate using local routes and any other explicitly configured private routes.

**Important:** a route does not override security controls. Routing, IP addressing, Security Groups, NACLs, host firewalls, and the service itself must all permit the intended connection.

### Traffic-path exercise

For each scenario, write the expected path and list the controls to check:
1. Internet client → public web endpoint.
2. Private application instance → public package repository via NAT Gateway.
3. Application instance → database in a private subnet.
4. One VPC → another VPC over peering.

## 4. Security Groups vs Network ACLs

| Feature | Security Group | Network ACL |
|---|---|---|
| Scope | Resource/network-interface level | Subnet level |
| State | Stateful: response traffic is tracked | Stateless: inbound and outbound rules are evaluated independently |
| Rules | Allow rules | Allow and deny rules |
| Rule processing | All applicable rules are evaluated together | Rules are processed by rule number, lowest first |
| Return traffic | Automatically allowed for an established permitted flow | Must be allowed by the relevant inbound and outbound rules |

Practical rule of thumb: use restrictive Security Groups as the primary workload-level control, and use NACLs when subnet-level allow/deny filtering is required. Avoid opening SSH/RDP to the entire internet; use a tightly restricted source, Systems Manager Session Manager, or a controlled access path where appropriate.

## 5. VPC connectivity options

- **VPC peering:** private connectivity between two VPCs. Peering is not transitive; a third VPC does not automatically route through a peered VPC.
- **AWS Transit Gateway:** a central hub for connecting multiple VPCs and supported networks. It simplifies larger network topologies but requires deliberate route-table and attachment design.
- **Route 53:** AWS DNS service for domain registration, DNS routing, health checks, and private hosted zones. DNS resolution and network reachability are separate concerns: a successful DNS lookup does not prove that the target service is reachable.

Choose based on scale, routing requirements, isolation, operational complexity, and cost—not just the number of resources.

## 6. Hands-on lab — design a custom VPC

### Before creating resources
- Use a learning/sandbox AWS account and confirm the current pricing for NAT Gateway and any other billable services.
- Prefer a diagram and route-table plan before provisioning.
- Never commit access keys, passwords, private keys, or account-specific secrets to GitHub.
- Use a region you are allowed to use and clean up resources when finished.

### Lab tasks
1. Draw a VPC with a non-overlapping CIDR range.
2. Plan at least one public and one private subnet; for a highly available design, plan them across two Availability Zones.
3. Identify the route table associated with each subnet.
4. Explain where an IGW and (if required) a NAT Gateway belong.
5. Draft least-privilege Security Group rules for a web tier and an internal application tier.
6. Explain how the application tier would reach a database without exposing the database publicly.
7. Describe how you would connect a second VPC: peering for a simpler direct connection, or Transit Gateway for a hub-and-spoke design.
8. Verify DNS resolution and connectivity separately during testing.
9. Document the intended traffic paths and the cleanup steps.

### Optional implementation
If you provision the lab in AWS, use the console or Infrastructure as Code only after reviewing the plan. Do not expose a test VM to unrestricted inbound SSH/RDP. A NAT Gateway can incur ongoing charges; delete it and other lab resources when done, and verify that no billable resources remain.

## 7. Troubleshooting checklist

When a connection fails, check in this order:
1. Is the destination running and listening on the expected port?
2. Are the source and destination IP addresses correct?
3. Does DNS resolve to the expected address?
4. Are subnet associations and route-table entries correct in both directions?
5. Does the public subnet have an IGW route, and does the instance have the required public addressing?
6. For private outbound access, is the NAT Gateway available and correctly routed?
7. Do Security Group rules permit the flow and its required return traffic?
8. Do NACL inbound and outbound rules permit both directions?
9. Are host firewalls, application bindings, and network ranges correct?
10. Are VPC peering/TGW attachments active and routes configured on both sides?

## 8. DevOps connection

VPC design is foundational for secure deployments. Later, Terraform will let me version-control the VPC, subnets, route tables, and security rules, review changes through pull requests, and detect infrastructure drift. Good network design also supports private Kubernetes nodes, internal services, controlled egress, and separation between application and data tiers.

## 9. Interview practice

1. What is a VPC, and how is a subnet different from an Availability Zone?
2. What makes a subnet public?
3. What is the difference between an Internet Gateway and a NAT Gateway?
4. Why does a private subnet use a NAT Gateway?
5. Compare Security Groups and NACLs.
6. Why must NACL return traffic be configured separately?
7. Are VPC peering connections transitive?
8. When would you choose Transit Gateway over VPC peering?
9. What does Route 53 do, and why is DNS success not proof of network reachability?
10. An EC2 instance has no connectivity: how would you troubleshoot it methodically?

## 10. DSA practice

**Problem:** Binary Search in a sorted array.

- Target complexity: **O(log n)** time and **O(1)** extra space for an iterative implementation.
- Practice explaining the loop invariant and why the search interval shrinks on every iteration.

## Day 13 checkpoint

Before marking this session complete, I should be able to:
- [ ] Explain public vs private subnets without relying only on their names.
- [ ] Draw route paths for inbound web traffic and private outbound traffic.
- [ ] Explain stateful Security Groups vs stateless NACLs.
- [ ] Compare VPC peering, Transit Gateway, and Route 53.
- [ ] Produce a custom VPC diagram and a short troubleshooting runbook.
- [ ] Record what I built, what failed, how I diagnosed it, and what I learned.

**Next session:** Day 14 — Advanced AWS Services: ALB/NLB, Auto Scaling, multi-AZ high availability, S3 storage classes, and ECS.
