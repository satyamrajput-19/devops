# Day 7 — Networking Fundamentals

## Course Alignment

This follows **Session 7: Networking Fundamentals** from the DevOps & Cloud Engineering course plan.

Course topics:
- IP addressing, subnetting & CIDR
- LAN/WAN and how systems communicate
- DNS fundamentals; HTTP(S), ports & SSH
- Design a VPC subnet plan and visualize routing paths

## Learning Objectives

By the end of Day 7, I should be able to:
- Explain IPv4 addressing and identify network/host portions.
- Understand subnetting and CIDR notation.
- Distinguish LAN and WAN.
- Explain how systems communicate across networks.
- Understand the role of DNS.
- Explain HTTP and HTTPS at a practical level.
- Understand ports and common DevOps-related services.
- Explain SSH at a practical level.
- Design a basic VPC subnet plan and trace routing paths.

## 1. IP Addressing

An IP address identifies a network interface so systems can communicate across an IP network.

Example IPv4 address:

    192.168.1.10

IPv4 addresses contain 32 bits and are commonly written as four decimal octets.

A useful mental model:

    Network portion | Host portion

The subnet mask / CIDR prefix determines how much of the address belongs to the network.

Example:

    192.168.1.10/24

Here, /24 represents the network prefix length.

## 2. Subnetting

Subnetting divides a larger IP network into smaller logical networks.

Why subnet?
- Separate workloads
- Control routing
- Organize infrastructure
- Reduce unnecessary broadcast scope
- Build security boundaries

Example:

    10.0.0.0/24

A /24 IPv4 network contains 256 total addresses.

For practical cloud design, subnet planning should consider:
- Required number of networks
- Required host capacity
- Routing requirements
- Future infrastructure growth
- Public/private workload separation

## 3. CIDR

CIDR stands for Classless Inter-Domain Routing.

Examples:

    10.0.0.0/16
    10.0.1.0/24
    10.0.2.0/24

The /16 or /24 is the prefix length.

A smaller prefix length generally represents a larger address range.

A useful progression:

    /16  -> larger network
    /24  -> smaller network
    /28  -> much smaller network

CIDR is fundamental to VPC design, subnetting, routing and security rules.

## 4. LAN vs WAN

### LAN — Local Area Network

A LAN connects systems within a relatively local network environment.

Examples:
- Home network
- Office network
- Data-center network segment

### WAN — Wide Area Network

A WAN connects networks across larger geographic or organizational boundaries.

The Internet is the most familiar example of a large interconnected network.

DevOps relevance:

Cloud architectures frequently connect multiple networks, subnets, availability zones and external services.

## 5. How Systems Communicate

At a simplified level:

    Application
        |
        v
    Transport
        |
        v
      IP
        |
        v
    Network interface
        |
        v
      Network

For troubleshooting, determine where communication fails:

    DNS resolution
         |
         v
    IP connectivity
         |
         v
    Port reachability
         |
         v
    Protocol/application response

This layered troubleshooting mindset is important for DevOps.

## 6. DNS Fundamentals

DNS translates domain names into IP addresses and helps clients locate services.

Example:

    example.com
        |
        v
       DNS
        |
        v
    IP address

Useful command:

    nslookup example.com

On systems with dig installed:

    dig example.com

Basic troubleshooting questions:
1. Does the hostname resolve?
2. What IP address does it resolve to?
3. Can the destination IP be reached?
4. Is the required port reachable?
5. Is the application responding?

DNS issues can look like application outages, so DNS resolution should be checked early during troubleshooting.

## 7. HTTP and HTTPS

### HTTP

HTTP is an application-layer protocol used for communication between clients and web services.

Common port:

    80

### HTTPS

HTTPS is HTTP protected with TLS.

Common port:

    443

Typical request path:

    Client
      |
      v
    DNS resolution
      |
      v
    IP address
      |
      v
    TCP connection
      |
      v
    HTTP/HTTPS request
      |
      v
    Application

DevOps relevance:
- APIs
- Web applications
- Load balancers
- Ingress
- Health checks
- CI/CD endpoints
- Cloud services

## 8. Ports

A port helps identify a service endpoint on a host.

Common ports to recognize:

| Service | Common Port |
|---|---:|
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| DNS | 53 |
| RDP | 3389 |

Knowing ports is essential when debugging security groups, network security groups, firewalls, load balancers, Kubernetes services and cloud connectivity.

## 9. SSH

SSH provides secure remote access to systems.

Typical command:

    ssh username@server-ip

Example:

    ssh ubuntu@203.0.113.10

SSH commonly uses port 22.

For troubleshooting:

    ssh -v username@server-ip

Security practices:
- Use key-based authentication where appropriate.
- Protect private keys.
- Avoid exposing management ports unnecessarily.
- Restrict administrative access to trusted sources where possible.

## 10. Basic Network Troubleshooting Commands

Check local IP configuration:

    ip addr

Inspect routes:

    ip route

Test DNS resolution:

    nslookup example.com

or:

    dig example.com

Test connectivity:

    ping <ip-address>

Test a TCP port:

    nc -vz <host> <port>

Test an HTTP endpoint:

    curl -I https://example.com

Inspect a route:

    traceroute <host>

Depending on the operating system, the exact networking utilities available may differ.

## 11. VPC Subnet Planning

The course connects networking fundamentals to cloud networking through a VPC subnet plan.

Example conceptual design:

    VPC: 10.0.0.0/16

        |
        +-------------------+
        |                   |
        v                   v
    Public Subnet       Private Subnet
    10.0.1.0/24         10.0.2.0/24
        |                   |
        v                   v
    Internet-facing      Internal workloads

The exact architecture depends on workload and routing requirements.

When designing a subnet plan, document:
- VPC/network CIDR
- Subnet CIDRs
- Public/private role
- Route targets
- Workloads placed in each subnet
- Expected traffic paths

## 12. Visualizing Routing Paths

A routing path can be represented as:

    Client
      |
      v
    Internet
      |
      v
    Gateway / Load Balancer
      |
      v
    Public Subnet
      |
      v
    Private Subnet
      |
      v
    Application

For each hop, ask:
- What address is being used?
- What route sends the packet onward?
- Is a firewall/security rule involved?
- Which port is required?
- Is the destination service listening?

This is the foundation for cloud-network troubleshooting.

## 13. Hands-On Labs

### Lab 1 — IP and CIDR

Practice identifying network ranges from CIDR notation.

Tasks:
- Compare /16, /24 and /28.
- Identify whether two example addresses belong to the same subnet.
- Record your reasoning.

### Lab 2 — DNS + HTTP

Use:

    nslookup example.com
    curl -I https://example.com

Document:
- Resolved address
- HTTP response status
- What happened during DNS resolution

### Lab 3 — Port Troubleshooting

Choose a reachable service and test:

    nc -vz <host> <port>

Compare a reachable port with an intentionally incorrect port.

Document:
- Host
- Port
- Result
- Error/success message
- Your diagnosis

### Lab 4 — VPC Subnet Plan

Design:

    VPC: 10.0.0.0/16

Create a simple plan containing:
- Public subnet
- Private subnet
- CIDR ranges
- Intended workload
- Routing path

Draw the path from an external client to an internal application.

## 14. Failure Engineering

Intentionally investigate networking failures such as:
- Incorrect DNS name
- Incorrect IP address
- Closed port
- Service listening on the wrong port
- Missing route
- Incorrect subnet assumption
- HTTP vs HTTPS mismatch
- SSH connection failure

For every failure, answer:
1. What failed?
2. Was DNS resolution successful?
3. Was the IP reachable?
4. Was the required port reachable?
5. Was the service actually listening?
6. Which command revealed the problem?
7. How did I recover?

## 15. DevOps Connection

Networking is a dependency for almost every later DevOps topic.

    GitHub
       |
       v
    CI/CD
       |
       v
    Container Registry
       |
       v
    Cloud Network
       |
       v
    Kubernetes / Applications
       |
       v
    Monitoring

Later AWS, Docker, Kubernetes and GitOps work will depend on understanding:
- IP addressing
- CIDR
- Routing
- DNS
- Ports
- HTTP/HTTPS
- SSH
- Network security

## 16. Interview Questions

### Q1. What is an IP address?

An IP address identifies a network interface for communication using the Internet Protocol.

### Q2. What does /24 mean?

It indicates a 24-bit network prefix in CIDR notation.

### Q3. Why is subnetting used?

To divide a larger network into smaller logical networks for organization, routing and infrastructure design.

### Q4. What is DNS?

DNS maps domain names to network addresses and helps clients locate services.

### Q5. What is the difference between HTTP and HTTPS?

HTTPS adds TLS protection to HTTP communication.

### Q6. What is a port?

A port identifies a service endpoint on a host.

### Q7. What port does SSH commonly use?

TCP port 22.

### Q8. How would you troubleshoot an application that is unreachable?

Start by checking DNS resolution, then IP connectivity, route/path, port reachability, service status and application-level response.

### Q9. What is the purpose of a VPC subnet plan?

To organize cloud IP ranges, workloads and routing into a deliberate network design.

## 17. DSA / CS Fundamentals

### Problem — Binary Search

Given a sorted array and a target value, find the target's index or return -1 if it does not exist.

Example:

    Input:  [1, 3, 5, 7, 9], target = 7
    Output: 3

Target:
- Time: O(log n)
- Space: O(1)

Core idea:

Use the sorted property to repeatedly eliminate half of the remaining search space.

## 18. Day 7 Checkpoint

Before moving on, explain without notes:
- IPv4 addressing
- Subnetting
- CIDR
- LAN vs WAN
- How systems communicate
- DNS
- HTTP
- HTTPS
- Ports
- SSH
- Basic network troubleshooting
- VPC subnet planning
- Routing paths

Practical checkpoint:

Design a basic VPC subnet plan, document the CIDRs, identify public/private workloads, and draw the expected routing path from a client to an application.

## Learning Report

Record:
- One networking concept I understood well
- One networking command I practiced
- One failure I diagnosed
- One subnet/CIDR example I worked through
- One routing path I visualized
- One networking interview question I can now answer confidently

## Next

**Day 8 — Python for Automation I**

Course topics:
- Python syntax, data types, functions & error handling
- Working with files & directories (os, pathlib, shutil)
- Rewrite shell scripts in Python
- Virtual environments & pip
