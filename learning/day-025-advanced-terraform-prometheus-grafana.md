# Day 25 — Advanced Terraform + Prometheus & Grafana

## Today's goal

Session 25 builds on Terraform fundamentals with secure/scalable project structure, modules, workspaces, remote state, dynamic subnetting, infrastructure security checks, drift alerts, FinOps monitoring, and Prometheus/Grafana dashboards. The course also introduces Terragrunt, TFLint, and Checkov.

Today's output: a reusable Terraform VPC module design, a safe understanding of workspaces and remote state, and a local observability stack you can run and inspect.

## 7-hour plan

| Time | Focus | Deliverable |
|---|---|---|
| Hour 1 | Modules and reusable Terraform structure | Root/module design |
| Hour 2 | Workspaces and environment separation | Understand state isolation limits |
| Hour 3 | Remote state and locking | Backend design notes |
| Hour 4 | Dynamic subnetting and AWS security | CIDR calculations and validation |
| Hour 5 | TFLint and Checkov | Static checks and findings review |
| Hour 6 | Prometheus and Grafana | Local metrics dashboard |
| Hour 7 | Drift/FinOps plan, README, interview practice | Documented portfolio evidence |

## 1. Terraform modules

A module is a collection of Terraform files managed together. The root module calls child modules with input variables and receives outputs.

Suggested layout:

```text
terraform/
└── day-25/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── versions.tf
    └── modules/
        └── vpc/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf
```

Keep the module focused and reusable. Avoid hard-coding environment-specific values inside the child module.

Example child module `modules/vpc/variables.tf`:

```hcl
variable "name" {
  type        = string
  description = "Name prefix for the VPC"
}

variable "cidr_block" {
  type        = string
  description = "IPv4 CIDR block for the VPC"
}

variable "tags" {
  type        = map(string)
  default     = {}
}
```

Example child module `modules/vpc/main.tf`:

```hcl
resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = merge(var.tags, { Name = var.name })
}
```

Example child module `modules/vpc/outputs.tf`:

```hcl
output "vpc_id" {
  description = "ID of the created VPC"
  value       = aws_vpc.this.id
}

output "cidr_block" {
  description = "VPC IPv4 CIDR block"
  value       = aws_vpc.this.cidr_block
}
```

In the root module, call it from `main.tf`:

```hcl
module "vpc" {
  source     = "./modules/vpc"
  name       = "day25-lab-vpc"
  cidr_block = var.vpc_cidr

  tags = {
    Environment = terraform.workspace
    ManagedBy   = "Terraform"
  }
}
```

The root module must define `variable "vpc_cidr"`. A child module's output can be referenced as `module.vpc.vpc_id`.

## 2. Workspaces: what they do and do not do

Terraform CLI workspaces maintain separate state instances for the same configuration. They can be useful for lightweight parallel copies of the same infrastructure, but they are **not a complete security boundary** and do not automatically provide safe production isolation.

Useful commands:

```bash
terraform workspace show
terraform workspace list
terraform workspace new dev
terraform workspace select dev
```

Use separate AWS accounts, distinct backend/state arrangements, IAM boundaries, and protected deployment processes when environments need strong isolation. Be careful: applying in the wrong workspace or account can create or change the wrong infrastructure.

For production systems, explicit environment directories and separately controlled state can be clearer than using workspaces as the only separation mechanism.

## 3. Remote state and locking

Local state is fine for a small solo lab, but teams need a controlled shared state location. For AWS, a common pattern is an S3 backend with encryption, versioning, restricted access, and state locking supported by the chosen Terraform version/backend configuration.

Backend configuration is initialized separately from ordinary resource configuration. Example shape (configure the bucket and key only after you have created and secured the backend bucket):

```hcl
terraform {
  backend "s3" {
    bucket       = "REPLACE_WITH_YOUR_STATE_BUCKET"
    key          = "day25/network/terraform.tfstate"
    region       = "ap-south-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

This is a configuration example, not a ready-to-apply backend. The bucket must already exist, be private, and have appropriate versioning and access controls. Verify the locking configuration against your Terraform version. Never put credentials in the backend block. Backend changes generally require `terraform init -migrate-state` or reinitialization; read the prompts and back up state first.

**State protection checklist**
- Restrict access using least-privilege IAM.
- Enable bucket versioning and encryption.
- Keep state out of Git and public artifacts.
- Avoid concurrent operations without supported locking.
- Back up and plan state migrations carefully.
- Use separate state keys and permissions for isolated environments.

## 4. Dynamic subnetting with CIDR

Terraform functions such as `cidrsubnet` help calculate subnet ranges programmatically.

Example:

```hcl
variable "vpc_cidr" {
  type    = string
  default = "10.25.0.0/16"
}

locals {
  public_subnet_cidrs = [
    for netnum in range(2) : cidrsubnet(var.vpc_cidr, 8, netnum)
  ]
}

output "public_subnet_cidrs" {
  value = local.public_subnet_cidrs
}
```

For a `10.25.0.0/16` VPC and `newbits = 8`, the first two subnet CIDRs are `10.25.0.0/24` and `10.25.1.0/24`. Ensure subnet ranges do not overlap and fit within the VPC CIDR. For multi-AZ designs, map subnets deliberately to availability zones and ensure route/security rules match the intended public/private design.

## 5. TFLint and Checkov

These tools help catch problems before deployment:
- **TFLint:** Terraform-focused linting and provider-specific checks.
- **Checkov:** policy-as-code scanning for misconfigurations across IaC and cloud resources.
- **Terragrunt:** an orchestration/tooling layer that can help manage Terraform configurations across environments and modules; it is not a replacement for understanding Terraform itself.

Install tools using their official instructions for your operating system. Run the commands supported by your installed versions. Common command patterns:

```bash
terraform fmt -recursive
terraform validate
tflint --init
tflint
checkov -d .
```

Read each finding, understand its risk, and remediate or document a justified exception. Do not blindly suppress security findings just to get a clean report.

## 6. Local observability with Prometheus and Grafana

Prometheus collects and stores time-series metrics. Grafana queries data sources such as Prometheus and presents dashboards and alerts. For a first lab, run a local stack with Docker Compose.

Create a directory `observability/day-25/` with the following files.

### docker-compose.yml

```yaml
services:
  prometheus:
    image: prom/prometheus:v3.2.1
    ports:
      - "127.0.0.1:9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
      - node-exporter

  node-exporter:
    image: prom/node-exporter:v1.9.0
    ports:
      - "127.0.0.1:9100:9100"

  grafana:
    image: grafana/grafana:11.5.2
    ports:
      - "127.0.0.1:3000:3000"
    depends_on:
      - prometheus
```

These image versions are illustrative; check vendor documentation for current supported releases before using the stack beyond a local lab. Binding ports to 127.0.0.1 keeps these demo services local to the machine rather than listening on all network interfaces.

### prometheus.yml

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: node-exporter
    static_configs:
      - targets: ["node-exporter:9100"]
```

Start the stack:

```bash
docker compose up -d
docker compose ps
```

Open:
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000`

In Grafana, sign in to the local lab and add a Prometheus data source with URL `http://prometheus:9090` (the service name resolves inside the Compose network). Then explore metrics such as `up` and `node_load1`. Configure dashboards and alert rules deliberately; the demo is not a production monitoring configuration.

Stop the lab when finished:

```bash
docker compose down
```

For persistent Grafana dashboards, configure a named volume and a backup strategy. Do not expose Grafana publicly with default or weak credentials.

## 7. Drift alerts and FinOps monitoring

A Terraform plan can help identify drift, but it is not itself a complete alerting system. A practical approach is to run a scheduled, read-only plan in a controlled pipeline, capture the result, and notify the team when unexpected changes are detected. Ensure credentials are scoped and the plan output is handled as potentially sensitive.

FinOps checks to document:
- Resource tags for owner, environment, and purpose.
- AWS Budgets and cost alerts.
- Identify idle or oversized resources.
- Review NAT Gateway, load balancer, storage, and data-transfer costs.
- Remove temporary lab resources after testing.
- Avoid applying automated drift corrections to production without review.

Prometheus/Grafana generally monitor application and infrastructure metrics; AWS billing data is often best sourced from AWS billing/cost tools or exported datasets, then visualized in a dashboard if needed.

## 8. Troubleshooting

| Symptom | Check |
|---|---|
| Module input missing | Variable name/type and root module call |
| Output reference invalid | Output declared in child module and referenced as `module.NAME.output` |
| Wrong workspace/state | `terraform workspace show`, account, region, backend key |
| Backend initialization error | Bucket exists, IAM access, region, backend syntax, lock support |
| CIDR overlap | VPC range, subnet masks, `cidrsubnet` arguments |
| TFLint/Checkov findings | Read exact rule and fix or justify exception |
| Prometheus target down | Compose service name, scrape config, container logs |
| Grafana has no data | Prometheus data-source URL, target status, query and time range |
| Dashboard disappears | Persistence volume/provisioning and backup configuration |
| Unexpected cloud bill | Check resources, region, NAT/egress, logs, storage, and clean up |

## 9. Interview questions

1. Why use Terraform modules?
2. What do Terraform workspaces isolate, and what do they not isolate?
3. Why is remote state useful for a team?
4. How can you protect remote state?
5. Explain `cidrsubnet` and how you prevent overlapping subnet ranges.
6. How do TFLint and Checkov differ?
7. What is the role of Prometheus compared with Grafana?
8. What does the Prometheus `up` metric indicate?
9. How would you detect Terraform drift safely?
10. How would you build a cost-aware, observable cloud environment?

## 10. Completion checklist

- [ ] Create a reusable Terraform child module.
- [ ] Call the module from a root configuration and expose outputs.
- [ ] Practice workspace commands and understand their limitations.
- [ ] Document a secure remote state/backend design.
- [ ] Calculate subnet CIDRs and check for overlap.
- [ ] Run TFLint and Checkov, then review findings.
- [ ] Start Prometheus, node-exporter, and Grafana locally.
- [ ] Add Prometheus as a Grafana data source and query metrics.
- [ ] Document drift detection and FinOps controls.
- [ ] Stop local containers and clean up any cloud resources used.

**Next:** Day 26 — Getting Started with Ansible: configuration management, inventory, ad-hoc commands, roles, variables, facts, and idempotency.
