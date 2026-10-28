# Day 24 — Terraform: Introduction and State

## Today's goal

Session 24 introduces Infrastructure as Code (IaC), Terraform providers, resources, variables, outputs, state, drift detection, and a first AWS VPC configuration with subnets, route tables, and gateways.

By the end, you should be able to explain how Terraform compares declared configuration with tracked infrastructure, inspect a plan before making changes, and safely create and destroy a small networking lab.

> **Cost and safety:** This lab creates AWS networking resources only (VPC, subnet, Internet Gateway, route table, and route association). It does not create EC2 instances or a NAT Gateway. Still check your AWS account, region, and billing before applying. Never apply a configuration you have not reviewed.

## 7-hour study plan

| Time | Activity | Deliverable |
|---|---|---|
| 1 hour | IaC, Terraform workflow, providers/resources | Explain declarative infrastructure |
| 1 hour | Variables, outputs, expressions, resource references | Read and modify HCL |
| 1 hour | Terraform state, refresh, drift, locking | Explain why state matters |
| 2 hours | Build and validate a small AWS VPC configuration | Terraform files and reviewed plan |
| 1 hour | Apply, inspect, test, then destroy the lab | Evidence of lifecycle management |
| 1 hour | Troubleshooting, README, interview practice | GitHub documentation |

## 1. Core concepts

- **Infrastructure as Code (IaC):** describe infrastructure in version-controlled configuration rather than creating every resource manually.
- **Provider:** plugin that lets Terraform interact with an API, such as AWS.
- **Resource:** an infrastructure object Terraform manages, such as an AWS VPC.
- **Variable:** configurable input, such as the AWS region or CIDR block.
- **Output:** a value Terraform displays or makes available to other configurations, such as a VPC ID.
- **State:** Terraform's record mapping configuration resources to real infrastructure objects and their attributes.
- **Plan:** proposed changes Terraform calculates before applying.
- **Drift:** difference between the declared configuration/state and infrastructure changed outside Terraform.

## 2. Terraform workflow

```text
Write HCL -> terraform init -> terraform fmt
         -> terraform validate -> terraform plan
         -> review changes -> terraform apply
         -> inspect outputs/state -> terraform destroy (lab cleanup)
```

- `init` downloads providers and initializes the working directory.
- `fmt` formats configuration consistently.
- `validate` checks configuration syntax and internal consistency.
- `plan` previews intended changes; it does not create resources.
- `apply` executes the approved plan.
- `destroy` removes resources managed by the configuration. Review its plan carefully.

## 3. Install and authenticate

Install Terraform using the official HashiCorp installation instructions for your OS. Install and configure the AWS CLI as well.

Verify:

```bash
terraform version
aws --version
aws sts get-caller-identity
```

Use a dedicated AWS identity with only the permissions required for this lab. Prefer short-lived credentials or AWS IAM Identity Center where available. Do not paste access keys into Terraform files, commit them, or place them in GitHub.

Select your intended region explicitly. The configuration below uses `ap-south-1` (Mumbai); change it if needed and confirm your account can use it.

## 4. Create the project

Suggested folder: `terraform/day-24-vpc/`

```text
terraform/day-24-vpc/
├── versions.tf
├── variables.tf
├── main.tf
├── outputs.tf
└── .gitignore
```

### versions.tf

```hcl
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

The version constraint selects a compatible AWS provider 5.x release. In a real project, review provider upgrades and commit the generated dependency lock file ` .terraform.lock.hcl `.

### variables.tf

```hcl
variable "aws_region" {
  description = "AWS region for the lab"
  type        = string
  default     = "ap-south-1"
}

variable "project_name" {
  description = "Short prefix for lab resources"
  type        = string
  default     = "day24-terraform-lab"
}

variable "vpc_cidr" {
  description = "IPv4 CIDR range for the VPC"
  type        = string
  default     = "10.24.0.0/16"
}

variable "public_subnet_cidr" {
  description = "IPv4 CIDR range for the public subnet"
  type        = string
  default     = "10.24.1.0/24"
}
```

### main.tf

```hcl
resource "aws_vpc" "lab" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name        = "${var.project_name}-vpc"
    Environment = "learning"
    ManagedBy   = "Terraform"
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.lab.id
  cidr_block              = var.public_subnet_cidr
  map_public_ip_on_launch = true

  tags = {
    Name        = "${var.project_name}-public-subnet"
    Environment = "learning"
    ManagedBy   = "Terraform"
  }
}

resource "aws_internet_gateway" "lab" {
  vpc_id = aws_vpc.lab.id

  tags = {
    Name = "${var.project_name}-igw"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.lab.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.lab.id
  }

  tags = {
    Name = "${var.project_name}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

The route table points IPv4 internet-bound traffic to the Internet Gateway. A subnet is not safely reachable from the internet merely because it has a route: an instance, security group, network ACL, public addressing, and host configuration all affect connectivity. This lab does not launch an instance.

### outputs.tf

```hcl
output "vpc_id" {
  description = "ID of the lab VPC"
  value       = aws_vpc.lab.id
}

output "public_subnet_id" {
  description = "ID of the lab public subnet"
  value       = aws_subnet.public.id
}

output "internet_gateway_id" {
  description = "ID of the lab Internet Gateway"
  value       = aws_internet_gateway.lab.id
}
```

### .gitignore

```gitignore
.terraform/
*.tfstate
*.tfstate.*
crash.log
crash.*.log
*.tfplan
override.tf
override.tf.json
*_override.tf
*_override.tf.json
```

Do **not** ignore the provider dependency lock file. Do not commit state files: they can contain sensitive values. This lab uses local state for learning; a team project should evaluate a remote backend with access controls, encryption, versioning, and state locking appropriate to the chosen backend.

## 5. Initialize, validate, plan, apply

Run these commands from `terraform/day-24-vpc/`:

```bash
terraform init
terraform fmt -recursive
terraform validate
terraform plan -out=day24.tfplan
terraform show day24.tfplan
```

Read the plan. Confirm the region, CIDR blocks, resource names, and that no unexpected resources will be created or destroyed. If the plan is not what you expect, stop and troubleshoot.

Only when you are ready to create the lab resources:

```bash
terraform apply day24.tfplan
terraform output
terraform state list
```

Record the outputs (resource IDs only; never record credentials). Confirm the VPC and subnet appear in the correct AWS region.

## 6. Understand state and drift

Terraform state connects a resource address such as `aws_vpc.lab` to the real AWS VPC ID. It also stores attributes Terraform needs to calculate future changes.

Useful commands:

```bash
terraform state list
terraform state show aws_vpc.lab
terraform plan
```

If someone changes a resource manually in AWS, a later plan may detect a difference and propose updating the resource back to configuration or making another change. Do not assume every drift scenario is automatically harmless; inspect the proposed actions.

**State safety rules**
- Never edit state JSON by hand.
- Never share state files publicly.
- Avoid concurrent operations against the same local state.
- For teams, use a suitable remote backend with locking and controlled access.
- Back up and protect state before state migrations or recovery operations.

## 7. Cleanup

When finished, first inspect what Terraform proposes to remove:

```bash
terraform plan -destroy
```

If the plan contains only the resources you created for this lab, destroy them:

```bash
terraform destroy
```

Confirm the resources are gone in AWS. If you intend to continue using the lab, do not run destroy yet. Be especially careful when working in an account that contains other resources.

## 8. Troubleshooting

| Error or symptom | Check |
|---|---|
| Provider initialization fails | Internet/proxy access, provider source/version, rerun `terraform init` if configuration changed |
| AWS credentials error | `aws sts get-caller-identity`, credential profile, expired session |
| Access denied | IAM permissions and explicit denies/SCPs |
| Invalid CIDR or overlap | CIDR syntax and subnet range within the VPC range |
| Region/resource not found | AWS CLI profile and Terraform region |
| Plan proposes unexpected replacement | Resource arguments, provider behavior, and the full plan before applying |
| State lock issue | Whether another Terraform process is active; do not forcibly unlock without verifying ownership |
| Resource remains after destroy | Check the correct workspace, state, account, region, and whether resources were created outside this state |

## 9. Interview questions

1. What is Infrastructure as Code, and why use Terraform?
2. What is the difference between a provider and a resource?
3. Explain `terraform init`, `plan`, `apply`, and `destroy`.
4. What is Terraform state and why is it required?
5. What is infrastructure drift, and how do you investigate it?
6. Why should state files not be committed to Git?
7. What are variables and outputs?
8. Why review a plan before applying?
9. What is the purpose of an Internet Gateway and a route table in this lab?
10. How would you share state safely across a team?

## 10. Completion checklist

- [ ] Terraform and AWS CLI installed and version checked.
- [ ] Confirmed AWS identity and region.
- [ ] Created `versions.tf`, `variables.tf`, `main.tf`, and `outputs.tf`.
- [ ] Added a safe `.gitignore`.
- [ ] Ran `terraform init`, `fmt`, and `validate`.
- [ ] Reviewed the plan before applying.
- [ ] Created and inspected the VPC lab resources, if using AWS.
- [ ] Understood state and drift commands.
- [ ] Destroyed the lab resources or clearly documented that they remain.
- [ ] Added the lesson and evidence to GitHub without credentials or state files.

**Next:** Day 25 — Advanced Terraform, modules, workspaces, remote state, plus Prometheus and Grafana.
