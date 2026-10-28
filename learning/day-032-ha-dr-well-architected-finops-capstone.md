# Day 32 — HA, DR, AWS Well-Architected, FinOps & Career Capstone

## Course scope
The course's Session 32 covers high availability (HA) and disaster recovery (DR) across multiple Availability Zones/Regions, the six AWS Well-Architected pillars, cost optimization and FinOps (budgets, right-sizing and Savings Plans), AI-assisted DevOps, production architecture design, and combined DevOps/cloud interview preparation. The capstone is to design, defend and cost a full production architecture end-to-end.

## Learning outcomes
- Distinguish high availability from disaster recovery.
- Choose a recovery strategy using business impact, RTO and RPO.
- Explain the six AWS Well-Architected pillars.
- Build a practical cost-control and FinOps checklist.
- Present an architecture with security, resilience, observability and trade-offs.
- Practise interview questions and turn the course portfolio into a job-search asset.

> **Cost warning:** This is primarily an architecture/design and estimation lab. Multi-AZ resources, NAT Gateways, load balancers, databases, EKS, cross-region replication and data transfer can create charges. Do not deploy resources unless you have approval, a budget and a cleanup plan. Never commit AWS credentials, Terraform state containing secrets, private keys or real secret values.

## 7-hour plan
1. **Hour 1:** HA vs DR, failure domains, RTO/RPO.
2. **Hour 2:** Multi-AZ and multi-Region architecture choices.
3. **Hour 3:** Six Well-Architected pillars and design review.
4. **Hour 4:** FinOps, cost estimates, budgets and right-sizing.
5. **Hour 5:** AI-assisted DevOps with verification and security checks.
6. **Hour 6:** Write and diagram the end-to-end capstone.
7. **Hour 7:** Architecture defense, interview questions and portfolio review.

## 1. High availability vs disaster recovery

- **High availability (HA):** Minimize service interruption during common failures, such as an instance or Availability Zone outage.
- **Disaster recovery (DR):** Restore service after a larger disruptive event, such as regional failure, major data corruption or a security incident.

Two essential objectives:
- **RTO (Recovery Time Objective):** Maximum target time to restore a service.
- **RPO (Recovery Point Objective):** Maximum target amount of data loss measured in time.

Example: RTO = 1 hour and RPO = 15 minutes means the design aims to restore service within an hour and limit lost updates to roughly the previous 15 minutes. These are business targets, not automatic AWS guarantees.

Common DR patterns, from generally lower to higher standby cost and often faster recovery:
1. **Backup and restore:** Restore infrastructure and data from backups after a disruption.
2. **Pilot light:** Keep critical core components/data ready; scale out during recovery.
3. **Warm standby:** Keep a reduced but functioning environment running.
4. **Multi-site active/active:** Run traffic in multiple sites/Regions, with greater complexity and cost.

Choose a pattern from the required RTO/RPO, workload dependencies, data consistency, operational capacity and budget—not from complexity alone.

## 2. Reference architecture to defend

For a typical customer-facing web application, a reasonable starting point is:

    Users
      |
    DNS + TLS
      |
    Load balancer across multiple Availability Zones
      |
    Stateless application tier across multiple AZs
      |
    Managed database with Multi-AZ configuration
      |
    Backups / snapshots / tested restore process

    CI/CD pipeline -> security checks -> artifact registry
                         |
    GitOps / IaC ------> deployment and infrastructure changes

    Logs + metrics + alarms + audit trail
    Budget alerts + cost allocation tags
    Cross-Region backup/replication when business RTO/RPO justify it

This is a reference pattern, not a complete deployment blueprint. Select actual services based on requirements. For Kubernetes workloads, add cluster/node availability, pod spread, readiness/liveness probes, autoscaling, ingress, secret handling and persistent storage considerations. Multi-AZ availability alone does not protect against accidental deletion, credential compromise, data corruption or a regional outage.

### Design-review questions
- What happens if one instance, one AZ or the Region fails?
- Which components are stateful? How is data backed up and restored?
- Have restore procedures been tested?
- What are the RTO/RPO targets, and how are they measured?
- Can the system scale during a traffic spike?
- How are secrets, identities, network access and encryption managed?
- How are deployments rolled back?
- What alerts identify customer impact?
- What is the monthly cost estimate and largest cost uncertainty?
- Who owns incident response and DR exercises?

## 3. AWS Well-Architected Framework — six pillars

1. **Operational Excellence:** Operate and improve systems; runbooks, observability, incident reviews and automation.
2. **Security:** Strong identity controls, least privilege, traceability, encryption and protection at every layer.
3. **Reliability:** Recover from failure, test recovery, manage change and scale capacity.
4. **Performance Efficiency:** Select suitable resources and evolve designs as demand changes.
5. **Cost Optimization:** Measure spend, avoid waste, choose fit-for-purpose resources and optimize continuously.
6. **Sustainability:** Improve resource utilization and reduce unnecessary consumption and environmental impact.

Use the pillars as review lenses. One decision may improve one pillar while creating a trade-off in another. For example, multi-Region active/active may improve resilience but increase cost and operational complexity.

## 4. FinOps and cost optimization

A useful monthly review:
1. Inspect AWS Cost Explorer and the Cost and Usage Report if enabled.
2. Group spending by service, account, environment and cost-allocation tags.
3. Identify idle or oversized compute, unattached storage and unused load balancers/IP resources.
4. Review NAT Gateway, data transfer, logging and storage lifecycle costs.
5. Compare On-Demand usage with commitment options such as Savings Plans only after understanding stable usage and terms.
6. Set AWS Budgets and alert thresholds; remember that budgets alert rather than automatically guaranteeing a hard spending cap.
7. Forecast expected spend and explain material variances.
8. Record an owner, expected saving, risk and follow-up date for each optimization.

**Cost-estimation exercise:** Create a spreadsheet with rows for compute, database, load balancing, NAT/data transfer, storage, logs/monitoring, backup/replication and support. For each, note quantity, operating hours, unit-price source, monthly estimate, assumptions and confidence level. Use the AWS Pricing Calculator and current regional prices for real estimates; do not invent a precise monthly bill without inputs.

Trade-offs to explain:
- Multi-AZ improves resilience but may increase cost.
- Cross-Region copies improve recovery options but add storage, transfer and operations costs.
- Autoscaling helps match capacity to demand but needs limits and monitoring.
- Savings Plans may reduce eligible compute costs but involve a commitment.
- Logs are useful for security and troubleshooting but require retention and cost controls.

## 5. AI-assisted DevOps — use responsibly

AI can help draft shell scripts, Terraform, Kubernetes YAML, CI/CD workflows, runbooks and troubleshooting hypotheses. Treat output as an untrusted draft:
- Never submit access keys, customer data, private code or secrets to an unapproved model/tool.
- Review commands for destructive behavior before execution.
- Validate Terraform with fmt/validate/plan and review the plan before apply.
- Lint and scan manifests/code; test in a disposable environment.
- Verify API versions and current documentation.
- Ask for assumptions and failure modes, not just code.
- Keep a human accountable for production changes.

## 6. Capstone deliverables

Create a folder such as capstone/ in your portfolio repository with these files:

    capstone/
      README.md
      architecture.md
      diagrams/
        architecture.png
      cost-estimate.md
      disaster-recovery.md
      security-review.md
      operations-runbook.md

Your README should state:
- Problem statement, users and key requirements.
- Availability target, RTO and RPO.
- Architecture diagram and component responsibilities.
- Security boundaries and identity model.
- CI/CD and GitOps flow.
- Monitoring, alerting and incident response.
- Backup, restore and DR testing plan.
- Cost estimate, assumptions and optimization choices.
- Trade-offs, risks, limitations and next improvements.

Do not claim to have deployed or load-tested a system unless you actually did so. Clearly label architecture proposals, estimates, local labs and verified cloud deployments separately.

## 7. Career capstone presentation

Prepare a 5–7 minute explanation:
1. **Problem:** What workload are you designing for?
2. **Requirements:** Availability, latency, data protection, security and budget.
3. **Architecture:** Walk through traffic, compute, data, deployment and observability.
4. **Failure handling:** Explain instance/AZ/Region failure and recovery.
5. **Security:** Least privilege, secrets, network controls, encryption and audit.
6. **Cost:** State assumptions, major cost drivers and optimization choices.
7. **Trade-offs:** Explain what you intentionally did not build and why.
8. **Evidence:** Link to code, diagrams, pipeline results and test notes.

For entry-level cloud/DevOps roles, prioritize demonstrating fundamentals clearly: Linux, networking, Git, scripting, AWS IAM/VPC/EC2/S3/monitoring, Docker, CI/CD, Terraform, Kubernetes, troubleshooting and secure practices. Be honest about what you have implemented independently.

## Interview questions

1. **HA vs DR?** HA reduces interruption during common failures; DR restores service after larger disruptive events.
2. **RTO vs RPO?** RTO is the target recovery time; RPO is the target acceptable data-loss window.
3. **Does Multi-AZ equal DR?** Not by itself. It improves availability within a Region but does not cover every regional, deletion or corruption scenario.
4. **What is a DR drill?** A controlled exercise validating recovery steps, dependencies, access and whether measured RTO/RPO meet targets.
5. **Name the six Well-Architected pillars.** Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization and Sustainability.
6. **How would you reduce AWS costs?** Measure and allocate spend, remove waste, right-size, scale to demand, optimize storage/transfer, and evaluate commitment discounts against stable usage.
7. **What does an AWS Budget do?** Tracks spend/usage against configured thresholds and can notify stakeholders; it is not automatically a hard spending limit.
8. **How do you make a deployment safer?** Automated tests/scans, immutable artifacts, least privilege, staged promotion, health checks, observability and a tested rollback strategy.
9. **How should AI-generated infrastructure code be handled?** Review, validate, scan and test it; inspect destructive operations and never expose secrets.
10. **How would you defend a design choice?** Tie it to explicit requirements, show alternatives, quantify cost/risk where possible, and explain residual risks.

## Final checklist
- [ ] Explain HA, DR, RTO and RPO.
- [ ] Compare backup/restore, pilot light, warm standby and active/active.
- [ ] Name and apply the six Well-Architected pillars.
- [ ] Prepare a cost estimate with explicit assumptions.
- [ ] Design budgets, alerts, tagging and right-sizing actions.
- [ ] Explain secure use of AI-assisted DevOps.
- [ ] Produce architecture, DR, security, cost and runbook documents.
- [ ] Practise the 5–7 minute architecture defense.
- [ ] Review GitHub projects and be ready to explain your own contributions.
- [ ] Confirm all temporary cloud resources are accounted for and cleaned up safely.

## Course completion: what next?
Finishing the 32-session plan is a milestone, not proof that every tool is mastered. Review your portfolio, run the labs you can safely afford, fix weak fundamentals, practise interviews, and begin applying to suitable direct-entry cloud/DevOps roles while continuing hands-on practice.
