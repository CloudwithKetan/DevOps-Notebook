# 🏗️ Terraform — Complete Notes



![Terraform Workflow](../../images/terraform-workflow.svg)

> Terraform is an Infrastructure as Code (IaC) tool by HashiCorp that lets you define cloud infrastructure in human-readable configuration files, then create, update, and destroy that infrastructure with commands — instead of clicking through cloud consoles.

---

## 1. Why Terraform? The Problem It Solves

Without IaC, infrastructure is created manually — clicking through AWS console, running CLI commands, SSHing into servers. This leads to:

- **Snowflake servers** — every environment slightly different, impossible to reproduce
- **No audit trail** — who changed what, when, and why?
- **Slow provisioning** — takes hours to set up a new environment
- **Human error** — missed steps, wrong config

Terraform lets you write your infrastructure like code. The same code that creates your dev environment creates production — identical, every time. It lives in Git, so you get version history, code review, and rollback.

**"Treat your infrastructure like software."**

---

## 2. How Terraform Works — The Core Workflow

```
Write  →  Plan  →  Apply  →  Destroy
```

1. **Write** — you describe what you want in `.tf` files (desired state)
2. **Plan** — Terraform compares desired state with current state and shows you exactly what it will create, change, or destroy
3. **Apply** — Terraform executes the plan and talks to cloud provider APIs
4. **Destroy** — tear down everything Terraform manages (useful for non-prod environments)

---

## 3. Key Concepts

### 3.1 Providers

A **provider** is a plugin that knows how to talk to a specific platform (AWS, Azure, GCP, Kubernetes, GitHub, etc.). Terraform itself doesn't know anything about AWS — the AWS provider does.

```hcl
# Configure the AWS provider
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"       # Use any 5.x version
    }
  }
  required_version = ">= 1.5.0"  # Minimum Terraform version
}

provider "aws" {
  region = "ap-south-1"  # Mumbai — close to Pune!
  
  # Credentials: set via environment variables or AWS CLI config
  # Never hardcode access keys in .tf files
}
```

Run `terraform init` to download the provider. It gets saved to `.terraform/` folder.

### 3.2 Resources

A **resource** is a single piece of infrastructure — an EC2 instance, an S3 bucket, a VPC. This is the core building block of Terraform.

```hcl
resource "<provider>_<type>" "<local_name>" {
  # configuration arguments
}
```

```hcl
# Create an EC2 instance
resource "aws_instance" "web_server" {
  ami           = "ami-0522ab6e1ddcc7055"   # Ubuntu 22.04 in ap-south-1
  instance_type = "t3.micro"

  tags = {
    Name        = "WebServer"
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}

# Create an S3 bucket
resource "aws_s3_bucket" "app_assets" {
  bucket = "my-company-app-assets"

  tags = {
    Name = "AppAssets"
  }
}
```

The `"web_server"` and `"app_assets"` are **local names** — they only matter inside your Terraform code, not in AWS.

### 3.3 Variables — Making Config Reusable

Instead of hardcoding values, use **input variables**. Define them in `variables.tf`, set values in `.tfvars` files or environment variables.

```hcl
# variables.tf
variable "environment" {
  description = "Deployment environment (dev, staging, prod)"
  type        = string
  default     = "dev"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Instance type must be t3.micro, t3.small, or t3.medium."
  }
}

variable "allowed_cidrs" {
  description = "List of CIDRs allowed to access the app"
  type        = list(string)
  default     = ["0.0.0.0/0"]
}
```

```hcl
# main.tf — using variables
resource "aws_instance" "web" {
  instance_type = var.instance_type

  tags = {
    Environment = var.environment
  }
}
```

```hcl
# terraform.tfvars — set values (do NOT commit secrets to Git!)
environment   = "production"
instance_type = "t3.medium"
```

### 3.4 Outputs

**Outputs** are how Terraform tells you information about the resources it created — like the IP address of a server or the ARN of a resource. They're also how modules pass data to each other.

```hcl
# outputs.tf
output "instance_public_ip" {
  description = "Public IP address of the web server"
  value       = aws_instance.web_server.public_ip
}

output "bucket_domain" {
  description = "S3 bucket domain name"
  value       = aws_s3_bucket.app_assets.bucket_domain_name
}
```

After `terraform apply`, these values are printed. Access them anytime with `terraform output`.

### 3.5 Data Sources

A **data source** reads existing resources — things Terraform didn't create but you need to reference. For example, fetching the latest Ubuntu AMI ID from AWS.

```hcl
# Find the latest Ubuntu 22.04 AMI automatically
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]   # Canonical's AWS account

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

# Use it in a resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id  # Always latest Ubuntu
  instance_type = "t3.micro"
}
```

Data sources are read-only — Terraform never modifies what a data source points to.

---

## 4. State — The Heart of Terraform

Terraform maintains a **state file** (`terraform.tfstate`) that records exactly which real resources it manages and their current configuration. This is how Terraform knows what already exists and what needs to change.

### Why state matters

When you run `terraform plan`, Terraform:
1. Reads your `.tf` files (desired state)
2. Reads the state file (last known real state)
3. Calls cloud APIs to verify actual current state
4. Shows you the difference (the plan)

### Remote State — Critical for Teams

By default, state is stored locally. **Never use local state in a team** — if two people run `terraform apply` simultaneously, they'll corrupt each other's state.

Store state remotely in an S3 bucket with DynamoDB locking:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    key            = "production/network/terraform.tfstate"
    region         = "ap-south-1"
    encrypt        = true                    # Encrypt at rest
    dynamodb_table = "terraform-state-lock"  # Prevents simultaneous applies
  }
}
```

**State locking** — when someone runs `terraform apply`, Terraform writes a lock to DynamoDB. Other runs must wait. This prevents race conditions.

### State Commands

```bash
# View what's in state
terraform state list

# Detailed info about one resource
terraform state show aws_instance.web_server

# Remove resource from state (without destroying it)
# Useful when you want Terraform to "forget" a resource
terraform state rm aws_instance.web_server

# Import an existing resource into Terraform management
terraform import aws_instance.web_server i-0abc123def456789
```

---

## 5. Modules — Reusable Infrastructure

A **module** is a container for multiple resources that are used together. Instead of writing the same VPC setup in every project, write it once as a module and reuse it everywhere.

```
project/
├── main.tf
├── variables.tf
├── outputs.tf
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── ec2/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

```hcl
# main.tf — calling a module
module "network" {
  source      = "./modules/vpc"      # Local module path
  
  vpc_cidr    = "10.0.0.0/16"
  environment = var.environment
  azs         = ["ap-south-1a", "ap-south-1b"]
}

module "webserver" {
  source      = "./modules/ec2"
  
  subnet_id   = module.network.public_subnet_ids[0]  # Output from vpc module
  environment = var.environment
}
```

You can also use modules from the **Terraform Registry** — a public library of pre-built modules for common infrastructure patterns:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b", "ap-south-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
}
```

---

## 6. Practical Example — Full AWS Setup

Here's a complete, realistic example: a VPC with public and private subnets, an EC2 web server, and a security group.

```hcl
# main.tf

# ── VPC ─────────────────────────────────────────────
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = { Name = "${var.project}-vpc" }
}

# ── Internet Gateway ─────────────────────────────────
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "${var.project}-igw" }
}

# ── Public Subnet ────────────────────────────────────
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true

  tags = { Name = "${var.project}-public" }
}

# ── Route Table for Public Subnet ────────────────────
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = { Name = "${var.project}-public-rt" }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# ── Security Group ───────────────────────────────────
resource "aws_security_group" "web" {
  name   = "${var.project}-web-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    description = "HTTP from anywhere"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS from anywhere"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"         # All outbound traffic allowed
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "${var.project}-web-sg" }
}

# ── EC2 Instance ─────────────────────────────────────
resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
    systemctl enable nginx
  EOF

  tags = {
    Name        = "${var.project}-webserver"
    Environment = var.environment
  }
}
```

Notice how resources **reference each other**: `aws_internet_gateway.main.id`, `aws_vpc.main.id`. Terraform uses these references to build a **dependency graph** and create resources in the right order.

---

## 7. Terraform Commands Reference

```bash
# ── Setup ────────────────────────────────────────────
terraform init          # Download providers and modules
terraform init -upgrade # Upgrade providers to latest allowed version

# ── Plan & Apply ─────────────────────────────────────
terraform plan                        # See what will change
terraform plan -out=tfplan            # Save plan to file
terraform apply                       # Apply with confirmation prompt
terraform apply tfplan                # Apply saved plan (no prompt)
terraform apply -auto-approve         # Skip confirmation (use in CI/CD)
terraform apply -target=aws_instance.web  # Apply only one resource

# ── Inspect ──────────────────────────────────────────
terraform show          # Show current state in readable format
terraform output        # Print all output values
terraform state list    # List all resources in state

# ── Destroy ──────────────────────────────────────────
terraform destroy                        # Destroy everything
terraform destroy -target=aws_instance.web  # Destroy one resource

# ── Formatting & Validation ───────────────────────────
terraform fmt           # Auto-format .tf files
terraform validate      # Check syntax and config validity

# ── Workspace (multiple environments) ─────────────────
terraform workspace new staging
terraform workspace select production
terraform workspace list
```

---

## 8. Terraform Best Practices

**Separate state per environment** — don't share state between dev, staging, and production. Use separate S3 keys or separate backends.

**Use workspaces or separate directories** — a common pattern is having `environments/dev/`, `environments/prod/` each with their own `main.tf` and `terraform.tfvars`.

**Lock provider versions** — specify exact version constraints to prevent unexpected upgrades from breaking your infra:
```hcl
version = "= 5.31.0"   # Exactly this version
version = "~> 5.31"    # Any 5.31.x patch
version = ">= 5.0, < 6.0"  # Range
```

**Never commit secrets** — use environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) or an IAM role. Never put credentials in `.tf` or `.tfvars` files.

**Use `terraform fmt` and `terraform validate` in CI** — catch formatting issues and errors before they hit production.

**Tag everything** — add consistent tags to every resource so you can track costs, filter in the console, and automate:
```hcl
locals {
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
    Owner       = "devops-team"
  }
}

resource "aws_instance" "web" {
  # ...
  tags = merge(local.common_tags, { Name = "WebServer" })
}
```

---

## 9. Terraform vs CloudFormation

| Feature | Terraform | CloudFormation |
|---------|-----------|----------------|
| Language | HCL (HashiCorp Config Language) | JSON or YAML |
| Multi-cloud | Yes — AWS, Azure, GCP, 3000+ providers | AWS only |
| State management | External (S3 + DynamoDB) | Managed by AWS |
| Modules | Terraform Registry + custom | Nested stacks |
| Drift detection | terraform plan | Drift detection tool |
| Community | Very large | Moderate |

Terraform is the industry standard for multi-cloud IaC. CloudFormation is fine if you're 100% AWS and want native integration.

---

*Source: Terraform Registry & HashiCorp Docs — https://registry.terraform.io/*
