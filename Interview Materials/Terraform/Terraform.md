# Terraform Interview Q&A

## Set 1: Terraform Basics & EC2

**1. How do you install Terraform on a Linux system?**
You can install Terraform by:
- Adding the HashiCorp GPG key and repository.
- Running `sudo apt update` and `sudo apt install terraform`.
- Verifying installation using `terraform -version`.

**2. What are the prerequisites for deploying an EC2 instance using Terraform?**
- Terraform installed
- AWS credentials configured (via environment variables or AWS CLI)
- Key pair created in AWS
- Valid AMI ID and instance type
- Proper IAM permissions for launching instances

**3. What does `terraform init` do?**
Initializes the current directory as a Terraform working directory. It:
- Downloads provider plugins
- Sets up the backend (if configured)
- Creates a `.terraform` directory

**4. What is the purpose of `terraform plan`?**
It shows a preview of the actions Terraform will perform when you apply the configuration. It compares the desired configuration with the current state.

**5. What is a Terraform state file?**
The `terraform.tfstate` file tracks the actual state of your infrastructure. It maps configuration resources to real-world resources. It is required for correct operation of `plan`, `apply`, and `destroy`.

**6. Why is the `.terraform` directory created?**
This directory contains:
- Downloaded provider plugins
- Module data
- Backend configurations

It helps manage dependencies and is essential for execution.

**7. What is the difference between `terraform apply` and `terraform destroy`?**
- `apply`: Provisions or updates infrastructure to match the configuration.
- `destroy`: Deletes all resources defined in the configuration.

**8. Can you explain the Terraform resource block syntax?**
```hcl
resource "<PROVIDER>_<RESOURCE_TYPE>" "<NAME>" {
  argument1 = "value"
  argument2 = "value"
}
```
Example:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abc123"
  instance_type = "t2.micro"
}
```

**9. What are output values in Terraform?**
Output values are returned to the user after a successful apply. They can expose resource attributes like instance IDs, IP addresses, etc.
```hcl
output "instance_id" {
  value = aws_instance.web.id
}
```

**10. What happens if you manually change a resource that Terraform manages?**
Terraform will detect the drift during `terraform plan` or `apply`, and may:
- Revert it to match the configuration
- Recreate the resource
- Show a plan to update it if possible

---

## Set 2: Security Groups, Variables & Data Sources

**1. What is a Security Group in AWS and how is it defined in Terraform?**
A Security Group is a virtual firewall that controls inbound and outbound traffic to AWS resources. In Terraform, it's defined using the `aws_security_group` resource block where we configure rules using `ingress` and `egress` blocks.

**2. How do you allow multiple ports in a single security group in Terraform?**
You can define multiple `ingress` or `egress` blocks inside a single `aws_security_group` resource, each specifying different port ranges and protocols.

**3. What is the HEREDOC syntax in Terraform? Why is it useful?**
HEREDOC (`<<EOF ... EOF`) allows defining multiline strings, typically used in `user_data` for EC2 initialization scripts. It improves readability and avoids complex escape sequences.

**4. How does `user_data` work in Terraform with EC2 instances?**
The `user_data` attribute sends a shell script to the instance, which is executed during the boot process. It's often used for installing software, configuring services, or automating setup.

**5. What is the purpose of the provider block in Terraform?**
The provider block specifies the platform (e.g., AWS, Azure, GCP) Terraform will interact with. It defines configuration like the region, credentials, and version constraints.

**6. What is the difference between `resource` and `data` blocks in Terraform?**
- `resource` creates and manages infrastructure.
- `data` retrieves and reads existing resources (without modifying them).

**7. Why are variables used in Terraform, and how are they defined?**
Variables allow dynamic and reusable configurations. They are defined using the `variable` block and can be passed through CLI, `.tfvars` files, or environment variables.

**8. What is the purpose of the output block in Terraform?**
`output` blocks display key values (like resource IDs or IPs) after an apply. They're useful for visibility, debugging, or passing data between modules.

**9. How can you reference one resource's output in another resource in Terraform?**
You can use interpolation syntax like `aws_instance.web.public_ip` to reference attributes of one resource inside another resource or output block.

**10. What happens if you make manual changes to an AWS resource outside Terraform?**
Terraform may detect configuration drift during `plan` or `apply`, and either:
- Revert the manual changes
- Mark the resource for recreation
- Apply updates if consistent with the config

---

## Set 3: Auto Scaling Groups & Load Balancers

**1. What is Terraform and how does it manage infrastructure?**
Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It allows users to define and provision data center infrastructure using a high-level configuration language. Terraform manages infrastructure using a state file and enables declarative deployments through `terraform plan` and `terraform apply`.

**2. What is an Auto Scaling Group (ASG) in AWS and why is it important?**
An Auto Scaling Group automatically adjusts the number of EC2 instances to meet demand. It helps maintain application availability and optimize costs by scaling in or out based on policies like CPU utilization, scheduled time, or demand.

**3. How does Terraform configure an Auto Scaling Group?**
Terraform configures an Auto Scaling Group using the `aws_autoscaling_group` resource. It can reference a `launch_template` or `launch_configuration` to define how new instances should be launched. The ASG configuration includes parameters like min size, max size, desired capacity, and health checks.

**4. What are the types of Load Balancers supported by AWS and Terraform?**
- **Application Load Balancer (ALB):** Operates at Layer 7 and supports path-based routing.
- **Network Load Balancer (NLB):** Operates at Layer 4 and is optimized for high performance.
- **Classic Load Balancer (CLB):** Legacy option that works at both Layer 4 and Layer 7.

**5. What is a Target Group in the context of Load Balancing?**
A Target Group is a set of endpoints (such as EC2 instances) that receive traffic from a Load Balancer. Each Target Group performs health checks and ensures traffic is only routed to healthy instances.

**6. How does Terraform handle dependencies between resources?**
Terraform automatically understands dependencies between resources based on how outputs and inputs are linked. For example, if an Auto Scaling Group references a Load Balancer's Target Group ARN, Terraform will create the Target Group first. The `depends_on` attribute can be used to define manual dependencies if needed.

**7. What is the significance of the `user_data` field in an EC2 launch template?**
The `user_data` field allows you to run startup scripts when an EC2 instance launches. This is typically used to install and configure software such as web servers or monitoring agents. In Terraform, the script is usually base64-encoded and passed inside the launch template.

**8. What is a typical configuration for an Auto Scaling Group in Terraform?**
A typical configuration includes:
- Minimum, maximum, and desired capacity values
- Launch template or configuration
- VPC subnets to launch instances into
- Health check type and grace period
- Target group ARN if using a Load Balancer

**9. How would you implement a Blue-Green deployment using Terraform?**
- Deploy two separate Auto Scaling Groups (Blue and Green).
- Attach only one to the Load Balancer initially (e.g., Blue).
- After testing the new version in the Green group, switch the Load Balancer to point to Green.
- Gradually terminate the Blue group after successful verification.

**10. What are the advantages of using Terraform for ASG and Load Balancer setup?**
- Infrastructure as Code allows version control and repeatability.
- Automated deployments reduce manual errors.
- Supports modular and reusable configurations.
- Works well with CI/CD pipelines.
- Provides visibility into infrastructure changes with the plan/apply workflow.

---

## Set 4: Terraform Concepts & Best Practices

**1. What is Terraform and how is it different from other IaC tools like Ansible or CloudFormation?**
- Compare declarative (Terraform) vs. procedural (Ansible).
- Terraform is cloud-agnostic; CloudFormation is AWS-specific.

**2. Explain the Terraform workflow.**
- `terraform init`
- `terraform plan`
- `terraform apply`
- `terraform destroy`

**3. What is the Terraform state file and why is it important?**
- Stores information about the infrastructure.
- Used for mapping resources to real-world infrastructure.
- Can be local or remote (e.g., in S3).

**4. What is the purpose of a backend in Terraform?**
- To store state remotely (e.g., S3 with DynamoDB for locking).
- Enables collaboration and consistency across teams.

**5. How do modules help in Terraform?**
- Promote code reuse and structure.
- Allow separation of infrastructure layers.
- Make it easier to manage and scale.

**6. How do you manage secrets in Terraform?**
- Never hard-code secrets in `.tf` files.
- Use environment variables, external tools like AWS Secrets Manager, or `terraform.tfvars` with `.gitignore`.

**7. How does Terraform handle resource dependencies?**
- Implicit via interpolation or direct reference.
- Explicit using `depends_on`.

**8. What is the difference between `count` and `for_each` in Terraform?**
- `count`: Indexed, best for identical resources.
- `for_each`: Key-value, better for mapping named resources.

**9. What happens if you manually delete a resource that Terraform manages?**
Terraform sees it as missing and will recreate it during the next apply.

**10. How do you handle environment-specific configurations in Terraform?**
- Use workspaces, separate variable files, or directory-based environments.
- Example: `dev.tfvars`, `prod.tfvars`

---

## Set 5: Comprehensive Terraform Q&A

**1. What is Terraform and how does it work?**
Terraform is an Infrastructure as Code (IaC) tool that allows users to define and provision infrastructure using a declarative configuration language. It manages infrastructure across multiple providers using a state file to track resources.

**2. What are providers in Terraform?**
Providers are plugins that interact with cloud platforms (like AWS, Azure, GCP). They allow Terraform to create, read, update, and delete resources on those platforms.

**3. What is the difference between Terraform and Ansible?**
- Terraform is declarative and used to provision infrastructure.
- Ansible is procedural and focuses on configuration management.
- Terraform is best for provisioning resources; Ansible is better for installing software and managing servers.

**4. Explain the Terraform workflow.**
- `terraform init`: Initializes the directory and downloads providers
- `terraform plan`: Shows what will be changed
- `terraform apply`: Applies the changes
- `terraform destroy`: Deletes the resources

**5. What is a Terraform module and why is it used?**
A module is a reusable group of Terraform resources. Modules promote code reuse, improve organization, and simplify managing complex environments.

**6. How does Terraform handle dependencies between resources?**
Terraform automatically detects implicit dependencies through references. For manual control, you can use `depends_on` to specify explicit dependencies.

**7. What is the purpose of the `terraform.tfstate` file?**
It keeps a snapshot of your infrastructure's current state. Terraform uses it to determine the difference between the current state and the desired configuration.

**8. What is remote state in Terraform?**
Remote state stores the state file in a shared backend (e.g., AWS S3, Terraform Cloud) to support collaboration, versioning, and state locking using DynamoDB.

**9. What are input and output variables in Terraform?**
- **Input variables** allow parameterization of Terraform configurations.
- **Output variables** expose resource values (like IP addresses) after apply.

**10. What is the difference between `count` and `for_each` in Terraform?**
- `count` is used for creating multiple identical resources using an index.
- `for_each` creates resources from maps or sets with named references.

**11. What are data sources in Terraform?**
Data sources allow you to fetch information defined outside Terraform (e.g., existing AMIs, VPCs) and use it in your configurations.

**12. How do you structure a multi-environment Terraform setup?**
- Use separate `.tfvars` files for each environment (e.g., dev, prod)
- Separate backend configurations
- Use workspaces or folder-based segregation

**13. What happens if a resource is deleted manually outside Terraform?**
Terraform will detect the missing resource during the next plan and recreate it to match the configuration.

# Terraform Interview Questions & Answers
> Covers Basics · Core Concepts · State · Modules · Workspaces · CI/CD · Security · Scenario-Based

---

## 🟢 SECTION 1: BASICS & CORE CONCEPTS

---

### 1. What is Terraform and why is it used?
**Answer:**
Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It lets you define, provision, and manage infrastructure across multiple cloud providers and services using a declarative configuration language called HCL (HashiCorp Configuration Language).

**Why it's used:**
- **Multi-cloud** — Works with AWS, Azure, GCP, Kubernetes, GitHub, Datadog, 1000+ providers
- **Declarative** — You describe the desired end state, not the steps to get there
- **Version control** — Infrastructure defined in code; tracked in Git
- **Idempotent** — Running the same config multiple times produces the same result
- **State management** — Tracks real-world resources vs desired state
- **Plan before apply** — Preview changes before making them (dry run)
- **Modular** — Reusable modules for common infrastructure patterns

---

### 2. What is Infrastructure as Code (IaC)?
**Answer:**
IaC is the practice of managing and provisioning computing infrastructure through machine-readable configuration files rather than through manual processes or interactive tools.

**Benefits:**
- **Consistency** — Same infrastructure every time, no human error
- **Speed** — Spin up full environments in minutes
- **Repeatability** — Identical environments for dev, staging, production
- **Documentation** — Config files serve as living documentation
- **Version control** — Audit trail of every infrastructure change
- **Cost visibility** — Know what you're running before deploying

**IaC Tools comparison:**
| Tool | Type | Scope |
|---|---|---|
| Terraform | Declarative | Multi-cloud, any provider |
| AWS CloudFormation | Declarative | AWS only |
| Ansible | Procedural | Config management + provisioning |
| Pulumi | Imperative (code) | Multi-cloud |
| ARM Templates | Declarative | Azure only |

---

### 3. What is HCL (HashiCorp Configuration Language)?
**Answer:**
HCL is the language used to write Terraform configuration files. It's designed to be both human-readable and machine-parseable.

```hcl
# Resource block
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name        = "web-server"
    Environment = "production"
  }
}

# Variable block
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Must be a valid t3 instance type."
  }
}

# Output block
output "instance_ip" {
  description = "Public IP of the web server"
  value       = aws_instance.web.public_ip
}
```

---

### 4. What is a Terraform Provider?
**Answer:**
A provider is a plugin that enables Terraform to interact with a specific API or service. Providers are responsible for understanding API interactions and exposing resources.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"      # Allow 5.x, not 6.x
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = ">= 2.0"
    }
    github = {
      source  = "integrations/github"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.5.0"
}

provider "aws" {
  region  = "us-east-1"
  profile = "production"

  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Environment = var.environment
    }
  }
}

# Multiple provider configurations (aliases)
provider "aws" {
  alias  = "eu"
  region = "eu-west-1"
}

resource "aws_instance" "eu_server" {
  provider = aws.eu
  ami      = "ami-..."
}
```

---

### 5. What are the core Terraform commands?
**Answer:**

```bash
# Initialization
terraform init                   # Initialize working directory, download providers & modules

# Planning
terraform plan                   # Preview changes (dry run)
terraform plan -out=tfplan       # Save plan to file
terraform plan -var="env=prod"   # Override variable

# Applying
terraform apply                  # Apply changes (prompts for confirmation)
terraform apply -auto-approve    # Apply without confirmation (CI/CD)
terraform apply tfplan           # Apply from saved plan file

# Destroying
terraform destroy                # Destroy all resources
terraform destroy -target=aws_instance.web   # Destroy specific resource

# State
terraform show                   # Show current state
terraform state list             # List all resources in state
terraform state show aws_instance.web       # Show specific resource
terraform state mv               # Move resource in state
terraform state rm               # Remove resource from state (stop managing)
terraform state pull             # Download remote state
terraform state push             # Upload local state

# Inspecting
terraform output                 # Show outputs
terraform output instance_ip     # Show specific output
terraform console                # Interactive expression evaluator
terraform validate               # Validate configuration syntax
terraform fmt                    # Format HCL files
terraform graph                  # Generate dependency graph (DOT format)

# Import
terraform import aws_instance.web i-1234567890abcdef0
```

---

### 6. What is `terraform init` and what does it do?
**Answer:**
`terraform init` initializes a Terraform working directory. It must be run before any other commands.

**What it does:**
1. **Downloads providers** — Fetches provider plugins specified in `required_providers`
2. **Downloads modules** — Fetches remote module sources
3. **Initializes backend** — Sets up state storage (local or remote)
4. **Creates `.terraform/` directory** — Stores provider binaries and module cache
5. **Creates `.terraform.lock.hcl`** — Records exact provider versions (commit this!)

```bash
terraform init                          # Standard init
terraform init -upgrade                 # Upgrade providers to latest matching version
terraform init -backend-config=prod.tfbackend  # Use specific backend config
terraform init -reconfigure             # Reconfigure backend (ignore existing state)
terraform init -migrate-state           # Migrate state to new backend
```

---

### 7. What is `terraform plan` and why is it important?
**Answer:**
`terraform plan` creates an execution plan — it reads the current state, compares it to the desired configuration, and shows exactly what changes will be made WITHOUT making any actual changes.

**Output symbols:**
- `+` — Resource will be created
- `-` — Resource will be destroyed
- `~` — Resource will be updated in-place
- `-/+` — Resource will be destroyed and recreated (replacement)
- `<=` — Data source will be read

```bash
terraform plan

# Output:
# Plan: 3 to add, 1 to change, 0 to destroy.
#
# + aws_instance.web
#   ami:           "ami-abc123"
#   instance_type: "t3.micro"
#
# ~ aws_security_group.web
#   description:   "Old description" → "New description"

# Save plan for later apply (guarantees same changes)
terraform plan -out=my.tfplan
terraform apply my.tfplan
```

---

### 8. What is Terraform State?
**Answer:**
Terraform state is a JSON file (`terraform.tfstate`) that maps your Terraform configuration to real-world infrastructure. It's the source of truth for what Terraform currently manages.

**What state stores:**
- Resource IDs (e.g., EC2 instance ID, RDS endpoint)
- Resource attributes (IP, ARN, name)
- Dependency graph
- Metadata (provider version, resource type)

**Why state matters:**
- Terraform compares desired config vs state to determine what changes to make
- Without state, Terraform can't know if a resource already exists
- State enables `plan` to show accurate diffs

```bash
# View state
terraform show
terraform state list
cat terraform.tfstate    # Never edit manually!
```

**Important:** State can contain sensitive data (database passwords, private keys). Protect it with encryption and access controls.

---

### 9. What are Terraform Variables and how are they defined?
**Answer:**

```hcl
# variables.tf
variable "region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "us-east-1"
}

variable "instance_count" {
  type    = number
  default = 2
}

variable "enable_monitoring" {
  type    = bool
  default = true
}

variable "allowed_cidr_blocks" {
  type    = list(string)
  default = ["10.0.0.0/8", "172.16.0.0/12"]
}

variable "tags" {
  type = map(string)
  default = {
    Team    = "platform"
    Project = "infra"
  }
}

variable "db_config" {
  type = object({
    engine        = string
    instance_class = string
    storage       = number
  })
}
```

**Providing variable values (precedence order, highest to lowest):**
```bash
# 1. Command line flag (highest precedence)
terraform apply -var="region=eu-west-1"

# 2. .tfvars file (auto-loaded: terraform.tfvars or *.auto.tfvars)
# terraform.tfvars
region         = "eu-west-1"
instance_count = 3

# 3. Environment variables
export TF_VAR_region="eu-west-1"
export TF_VAR_instance_count=3

# 4. Default value in variable block (lowest precedence)
```

---

### 10. What are Terraform Outputs?
**Answer:**
Outputs expose values from your Terraform configuration — making resource attributes available to users, other modules, or CI/CD pipelines.

```hcl
# outputs.tf
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.main.id
}

output "db_endpoint" {
  description = "RDS endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true    # Hides value in CLI output
}

output "instance_ips" {
  description = "Public IPs of all instances"
  value       = [for inst in aws_instance.web : inst.public_ip]
}

output "load_balancer_dns" {
  description = "ALB DNS name"
  value       = aws_lb.main.dns_name
}
```

```bash
terraform output                        # All outputs
terraform output vpc_id                 # Specific output
terraform output -json                  # Machine-readable JSON
terraform output -raw load_balancer_dns # Raw value (no quotes)
```

---

## 🔵 SECTION 2: STATE MANAGEMENT

---

### 11. What is Remote State and why is it important?
**Answer:**
By default, Terraform stores state locally in `terraform.tfstate`. Remote state stores this file in a shared backend (S3, Azure Blob, GCS, Terraform Cloud).

**Why remote state:**
- **Team collaboration** — Everyone works with the same state
- **State locking** — Prevents concurrent modifications
- **Security** — State stored with encryption and access controls
- **Backup** — Cloud storage provides versioning and durability
- **CI/CD** — Pipelines can access shared state

```hcl
# backend.tf — S3 backend (most common for AWS)
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/vpc/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"    # For state locking
  }
}

# Azure backend
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstate12345"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# Terraform Cloud backend
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "production"
    }
  }
}
```

---

### 12. What is State Locking?
**Answer:**
State locking prevents concurrent state operations that could corrupt the state file. When a `plan` or `apply` is running, Terraform acquires a lock so no other operation can modify state at the same time.

**How it works with DynamoDB (AWS):**
1. Terraform writes a lock entry to DynamoDB before modifying state
2. Other Terraform runs see the lock and wait or fail
3. Lock is released when the operation completes

```hcl
# Create the DynamoDB table for locking
resource "aws_dynamodb_table" "terraform_lock" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

```bash
# If a previous run crashed and left a stale lock
terraform force-unlock LOCK_ID   # Use with caution!
```

---

### 13. What is `terraform import`?
**Answer:**
`terraform import` brings existing infrastructure (created manually or outside Terraform) under Terraform management WITHOUT destroying and recreating it.

```bash
# Import existing AWS EC2 instance
terraform import aws_instance.web i-1234567890abcdef0

# Import existing S3 bucket
terraform import aws_s3_bucket.data my-existing-bucket

# Import existing RDS instance
terraform import aws_db_instance.main mydb-identifier

# Import existing security group
terraform import aws_security_group.web sg-0abc123def456789
```

**Process:**
1. Write the resource block in your `.tf` file first
2. Run `terraform import` to map it to state
3. Run `terraform plan` — it should show no changes if the config matches reality
4. Fix any drift between config and imported resource attributes

**Terraform 1.5+ — Import blocks (config-driven import):**
```hcl
import {
  to = aws_instance.web
  id = "i-1234567890abcdef0"
}
```

---

### 14. What is `terraform taint` and `terraform untaint`?
**Answer:**
`terraform taint` marks a resource for forced replacement on the next `apply` — even if its configuration hasn't changed. Useful when a resource is in a bad state.

```bash
# Mark for replacement (deprecated in Terraform 1.0+)
terraform taint aws_instance.web

# Preferred replacement (Terraform 1.0+)
terraform apply -replace="aws_instance.web"

# Remove taint
terraform untaint aws_instance.web
```

**Use cases:**
- EC2 instance became unresponsive but Terraform doesn't know
- Database got corrupted
- TLS certificate needs rotation
- Resource is in error state and needs recreation

---

### 15. What is `terraform state mv`?
**Answer:**
`terraform state mv` moves a resource within state — either to rename it or move it between modules. This avoids destroying and recreating the resource.

```bash
# Rename a resource (config renamed, need to update state)
terraform state mv aws_instance.old_name aws_instance.new_name

# Move resource into a module
terraform state mv aws_instance.web module.web_server.aws_instance.this

# Move resource out of a module
terraform state mv module.web_server.aws_instance.this aws_instance.web

# Move resource to another state file
terraform state mv -state-out=new.tfstate aws_instance.web aws_instance.web
```

---

### 16. What is the difference between `terraform destroy` and `terraform state rm`?
**Answer:**

| Command | Effect on Cloud Resource | Effect on State |
|---|---|---|
| `terraform destroy` | Deletes the resource | Removes from state |
| `terraform state rm` | NO effect on cloud resource | Removes from state only |

```bash
# Actually delete the resource
terraform destroy -target=aws_instance.old_server

# Stop managing a resource (keep it running, just remove from state)
terraform state rm aws_instance.old_server
# After this, Terraform no longer manages it — it won't appear in plan
```

Use `state rm` when you want to "hand off" a resource to another team/codebase without deleting it.

---

## 🟡 SECTION 3: MODULES

---

### 17. What is a Terraform Module?
**Answer:**
A module is a container for multiple resources that are used together. Every Terraform configuration is technically a module (the root module). Modules enable reusability, abstraction, and organization.

```
infrastructure/
├── main.tf           ← Root module
├── variables.tf
├── outputs.tf
└── modules/
    ├── vpc/          ← Child module
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── ec2/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── rds/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

```hcl
# Using a local module
module "vpc" {
  source = "./modules/vpc"

  cidr_block   = "10.0.0.0/16"
  environment  = var.environment
  region       = var.region
}

# Using a Terraform Registry module
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name    = "my-cluster"
  cluster_version = "1.28"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnet_ids
}

# Using a Git module source
module "security" {
  source = "git::https://github.com/org/tf-modules.git//security?ref=v2.1.0"

  environment = var.environment
}

# Accessing module outputs
resource "aws_instance" "app" {
  subnet_id = module.vpc.private_subnet_ids[0]
  vpc_security_group_ids = [module.security.app_sg_id]
}
```

---

### 18. What is the difference between root module and child module?
**Answer:**

**Root Module:**
- The directory where you run `terraform plan/apply`
- Contains the main `.tf` files
- Has direct access to providers
- Manages the top-level state

**Child Module:**
- Called by root or other modules via `module` block
- Receives inputs via variables
- Returns outputs to the calling module
- Can use providers from the calling module

```hcl
# Root module (main.tf)
module "networking" {
  source      = "./modules/networking"   # Child module
  environment = "production"
}

# Access child module output
output "vpc_id" {
  value = module.networking.vpc_id       # From child module's outputs.tf
}
```

---

### 19. How do you pass providers to child modules?
**Answer:**

```hcl
# Root module — define providers
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "eu_west"
  region = "eu-west-1"
}

# Pass provider to module
module "us_resources" {
  source = "./modules/resources"
  # Uses default aws provider implicitly
}

module "eu_resources" {
  source = "./modules/resources"
  providers = {
    aws = aws.eu_west    # Explicitly pass aliased provider
  }
}

# In child module (modules/resources/main.tf)
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

resource "aws_vpc" "this" {
  cidr_block = "10.0.0.0/16"
  # Uses whatever aws provider was passed in
}
```

---

### 20. What are Module Sources?
**Answer:**

```hcl
# 1. Local path
module "vpc" {
  source = "./modules/vpc"
}
module "vpc" {
  source = "../shared/modules/vpc"
}

# 2. Terraform Registry (public)
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"
}

# 3. GitHub
module "security" {
  source = "github.com/org/terraform-modules//security"
}

# 4. Git (generic) with version tag
module "app" {
  source = "git::https://github.com/org/modules.git//app?ref=v1.5.0"
}

# 5. Git over SSH
module "private" {
  source = "git::ssh://git@github.com/org/private-modules.git//vpc?ref=main"
}

# 6. S3 bucket
module "internal" {
  source = "s3::https://s3.amazonaws.com/my-tf-modules/vpc.zip"
}

# 7. HTTP URL
module "external" {
  source = "https://example.com/vpc-module.zip"
}
```

---

## 🟠 SECTION 4: EXPRESSIONS, FUNCTIONS & LOOPS

---

### 21. What are `locals` in Terraform?
**Answer:**
`locals` define named values that can be used multiple times within a module. Unlike variables, they can't be overridden from outside.

```hcl
locals {
  # Simple values
  environment = "production"
  app_name    = "my-app"

  # Computed values
  common_tags = {
    Environment = local.environment
    Application = local.app_name
    ManagedBy   = "Terraform"
    CostCenter  = "platform-team"
  }

  # Conditional
  instance_type = var.environment == "production" ? "t3.medium" : "t3.micro"

  # Derived from other resources
  bucket_name = "${var.project}-${var.environment}-${data.aws_region.current.name}"

  # List manipulation
  availability_zones = slice(data.aws_availability_zones.available.names, 0, 3)
}

resource "aws_instance" "web" {
  instance_type = local.instance_type
  tags          = local.common_tags
}
```

---

### 22. What are `count` and `for_each` in Terraform?
**Answer:**

**`count`** — Creates multiple instances of a resource using an integer.

```hcl
# Create 3 EC2 instances
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-abc123"
  instance_type = "t3.micro"

  tags = {
    Name = "web-${count.index}"
  }
}

# Conditional resource creation
resource "aws_cloudwatch_metric_alarm" "cpu" {
  count = var.enable_monitoring ? 1 : 0
  # ...
}

# Access: aws_instance.web[0], aws_instance.web[1], etc.
```

**`for_each`** — Creates instances from a map or set of strings. Better than count because resources are identified by key, not index.

```hcl
variable "users" {
  default = {
    alice = { role = "admin",    email = "alice@example.com" }
    bob   = { role = "viewer",   email = "bob@example.com"   }
    carol = { role = "developer", email = "carol@example.com" }
  }
}

resource "aws_iam_user" "users" {
  for_each = var.users
  name     = each.key    # alice, bob, carol
  tags = {
    Email = each.value.email
    Role  = each.value.role
  }
}

# for_each with a set of strings
resource "aws_subnet" "public" {
  for_each = toset(["us-east-1a", "us-east-1b", "us-east-1c"])

  vpc_id            = aws_vpc.main.id
  availability_zone = each.value
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, index(tolist(toset(["us-east-1a","us-east-1b","us-east-1c"])), each.value))
}

# Access: aws_iam_user.users["alice"], aws_iam_user.users["bob"]
```

**`count` vs `for_each`:**
| | `count` | `for_each` |
|---|---|---|
| Index type | Integer | String key |
| Remove middle item | Recreates all items after it | Only removes the specific item |
| Best for | Identical resources | Resources with unique configs |

---

### 23. What are `for` expressions in Terraform?
**Answer:**
`for` expressions transform collections — like list comprehensions.

```hcl
# List → List (transform)
variable "names" {
  default = ["alice", "bob", "carol"]
}

locals {
  upper_names = [for name in var.names : upper(name)]
  # ["ALICE", "BOB", "CAROL"]

  # With filtering (if clause)
  long_names = [for name in var.names : name if length(name) > 3]
  # ["alice", "carol"]
}

# List → Map
locals {
  name_lengths = {for name in var.names : name => length(name)}
  # { alice = 5, bob = 3, carol = 5 }
}

# Map → List
variable "servers" {
  default = {
    web = "t3.micro"
    db  = "t3.large"
    app = "t3.small"
  }
}

locals {
  server_names = [for k, v in var.servers : "${k}: ${v}"]
  # ["web: t3.micro", "db: t3.large", "app: t3.small"]
}

# Flatten nested lists
locals {
  all_ips = flatten([for sg in aws_security_group.web : sg.ingress[*].cidr_blocks])
}
```

---

### 24. What are common Terraform built-in functions?
**Answer:**

```hcl
# String functions
upper("hello")               # "HELLO"
lower("HELLO")               # "hello"
format("Hello, %s!", "World") # "Hello, World!"
trimspace("  hello  ")       # "hello"
replace("hello world", "world", "terraform")  # "hello terraform"
split(",", "a,b,c")          # ["a", "b", "c"]
join("-", ["a", "b", "c"])   # "a-b-c"
substr("hello", 0, 3)        # "hel"
contains(["a","b"], "a")     # true

# Numeric functions
max(1, 2, 3)                 # 3
min(1, 2, 3)                 # 1
abs(-5)                      # 5
ceil(1.2)                    # 2
floor(1.9)                   # 1

# Collection functions
length(["a","b","c"])        # 3
toset(["a","b","a"])         # {"a","b"} (dedup)
tolist(toset(["a","b"]))     # ["a","b"]
tomap({a=1, b=2})            # {a=1, b=2}
flatten([[1,2],[3,4]])        # [1,2,3,4]
merge({a=1}, {b=2})          # {a=1, b=2}
keys({a=1, b=2})             # ["a","b"]
values({a=1, b=2})           # [1, 2]
lookup({a=1, b=2}, "a", 0)   # 1
element(["a","b","c"], 1)    # "b"
slice(["a","b","c","d"], 1, 3)  # ["b","c"]
compact(["a","","b"])        # ["a","b"] (removes empty)
distinct(["a","b","a"])      # ["a","b"]

# IP/Network functions
cidrsubnet("10.0.0.0/16", 8, 1)   # "10.0.1.0/24"
cidrhost("10.0.0.0/24", 5)        # "10.0.0.5"
cidrnetmask("10.0.0.0/24")        # "255.255.255.0"

# Encoding
base64encode("hello")             # "aGVsbG8="
base64decode("aGVsbG8=")         # "hello"
jsonencode({a=1})                 # "{\"a\":1}"
jsondecode("{\"a\":1}")           # {a=1}
yamlencode({a=1})                 # "a: 1\n"

# Filesystem (during plan/apply)
file("./user_data.sh")            # Read file contents
templatefile("./template.tpl", {name = "world"})  # Template rendering
filebase64("./cert.pem")          # File as base64

# Type conversion
tostring(42)                      # "42"
tonumber("42")                    # 42
tobool("true")                    # true
```

---

### 25. What is a `data` source in Terraform?
**Answer:**
Data sources allow Terraform to **read** information from existing infrastructure or external sources without managing it. They're read-only and don't create resources.

```hcl
# Get latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Get existing VPC
data "aws_vpc" "existing" {
  tags = {
    Name = "production-vpc"
  }
}

# Get current AWS account info
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

# Get availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Read local file
data "local_file" "config" {
  filename = "${path.module}/config.json"
}

# Read from SSM Parameter Store
data "aws_ssm_parameter" "db_password" {
  name = "/prod/db/password"
}

# Use data sources
resource "aws_instance" "web" {
  ami               = data.aws_ami.amazon_linux.id
  instance_type     = "t3.micro"
  subnet_id         = data.aws_vpc.existing.id
  availability_zone = data.aws_availability_zones.available.names[0]

  tags = {
    AccountID   = data.aws_caller_identity.current.account_id
    Region      = data.aws_region.current.name
  }
}
```

---

## 🔴 SECTION 5: WORKSPACES & ENVIRONMENTS

---

### 26. What are Terraform Workspaces?
**Answer:**
Workspaces allow you to manage multiple state files within the same backend configuration. Each workspace is an isolated state. The default workspace is always named `default`.

```bash
# Workspace commands
terraform workspace list         # List all workspaces
terraform workspace new dev      # Create and switch to 'dev' workspace
terraform workspace select prod  # Switch to 'prod' workspace
terraform workspace show         # Show current workspace
terraform workspace delete dev   # Delete workspace (must be empty)
```

```hcl
# Use workspace name in configuration
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "production" ? "t3.medium" : "t3.micro"

  tags = {
    Environment = terraform.workspace
    Name        = "web-${terraform.workspace}"
  }
}

# Different variable files per workspace
# terraform.tfvars is auto-loaded
# But you can use:
terraform apply -var-file="envs/${terraform.workspace}.tfvars"
```

**Limitation of workspaces:** They share the same codebase and backend, making it hard to have truly different configurations. For production-grade environments, separate directories or separate backends are often better.

---

### 27. What are the different strategies for managing multiple environments?
**Answer:**

**Strategy 1: Workspaces (simple, shared code)**
```
state/
  default.tfstate
  dev.tfstate
  staging.tfstate
  production.tfstate
```

**Strategy 2: Directory-based (recommended for production)**
```
infra/
├── modules/           # Shared modules
│   ├── vpc/
│   └── eks/
├── environments/
│   ├── dev/
│   │   ├── main.tf    # References modules
│   │   ├── terraform.tfvars
│   │   └── backend.tf # Separate state per env
│   ├── staging/
│   └── production/
```

**Strategy 3: Terragrunt (DRY wrapper)**
```
infra/
├── modules/
└── live/
    ├── terragrunt.hcl    # Root config
    ├── dev/
    │   ├── vpc/
    │   │   └── terragrunt.hcl    # DRY, just sets vars
    │   └── eks/
    └── prod/
        ├── vpc/
        └── eks/
```

---

## 🟣 SECTION 6: ADVANCED CONCEPTS

---

### 28. What is `depends_on` in Terraform?
**Answer:**
`depends_on` explicitly declares a dependency between resources when Terraform can't automatically detect it (e.g., when the dependency is through behavior, not attribute references).

```hcl
resource "aws_iam_role_policy" "example" {
  name   = "example"
  role   = aws_iam_role.example.id
  policy = data.aws_iam_policy_document.example.json
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.app.id
  instance_type = "t3.micro"

  # Explicit dependency - even though no attribute reference
  # Ensures IAM role policy is attached before instance starts
  depends_on = [aws_iam_role_policy.example]
}

# Module-level depends_on
module "ecs_service" {
  source = "./modules/ecs"

  depends_on = [module.vpc, aws_iam_role.ecs_task]
}
```

---

### 29. What is `lifecycle` in Terraform?
**Answer:**
`lifecycle` customizes how Terraform handles resource creation, updates, and deletion.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"

  lifecycle {
    # Create new resource BEFORE destroying old one
    # Prevents downtime during replacement
    create_before_destroy = true

    # Ignore changes to specific attributes (managed outside Terraform)
    ignore_changes = [
      tags["LastUpdated"],
      user_data,            # Don't replace instance when user_data changes
      ami                   # Don't replace if AMI changes (handle via ASG)
    ]

    # Prevent accidental destruction
    prevent_destroy = true    # Terraform will error if you try to destroy

    # Custom condition (Terraform 1.2+)
    precondition {
      condition     = data.aws_ami.ubuntu.architecture == "x86_64"
      error_message = "AMI must be x86_64 architecture."
    }

    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must have a public IP."
    }
  }
}
```

---

### 30. What is `templatefile()` and how is it used?
**Answer:**
`templatefile()` reads a file and renders it as a template, substituting variable references with provided values. Used for user_data scripts, config files, etc.

```hcl
# Template file: templates/user_data.sh.tpl
# #!/bin/bash
# yum update -y
# amazon-linux-extras install docker
# systemctl start docker
# docker run -d -p 80:80 \
#   -e DB_HOST=${db_host} \
#   -e APP_ENV=${environment} \
#   ${docker_image}

resource "aws_instance" "app" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  user_data = templatefile("${path.module}/templates/user_data.sh.tpl", {
    db_host      = aws_db_instance.main.endpoint
    environment  = var.environment
    docker_image = "myapp:${var.app_version}"
  })
}
```

---

### 31. What is `dynamic` block in Terraform?
**Answer:**
`dynamic` blocks generate repeated nested blocks (like `ingress`, `egress`, `tag`) based on a collection. Avoids repeating similar blocks.

```hcl
variable "ingress_rules" {
  default = [
    { port = 80,  protocol = "tcp", cidr = "0.0.0.0/0",   description = "HTTP" },
    { port = 443, protocol = "tcp", cidr = "0.0.0.0/0",   description = "HTTPS" },
    { port = 22,  protocol = "tcp", cidr = "10.0.0.0/8",  description = "SSH from VPN" },
    { port = 8080, protocol = "tcp", cidr = "10.0.0.0/8", description = "App port" },
  ]
}

resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = [ingress.value.cidr]
      description = ingress.value.description
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

### 32. What is `terraform_remote_state` data source?
**Answer:**
`terraform_remote_state` reads the output values from another Terraform state file. Used to share data between separate Terraform configurations.

```hcl
# In networking stack (separate state)
output "vpc_id" {
  value = aws_vpc.main.id
}
output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}

# In application stack — reads networking outputs
data "terraform_remote_state" "networking" {
  backend = "s3"
  config = {
    bucket = "my-terraform-state"
    key    = "networking/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.networking.outputs.private_subnet_ids[0]
  vpc_security_group_ids = [aws_security_group.app.id]
}
```

---

### 33. What is `moved` block in Terraform?
**Answer:**
The `moved` block (introduced in Terraform 1.1) tells Terraform that a resource was renamed or moved to a module, without destroying and recreating it. An alternative to `terraform state mv` that lives in code.

```hcl
# Renamed resource: aws_instance.old → aws_instance.new
moved {
  from = aws_instance.old_name
  to   = aws_instance.new_name
}

# Moved into module
moved {
  from = aws_instance.web
  to   = module.web_server.aws_instance.this
}

# Moved between modules
moved {
  from = module.old_module.aws_s3_bucket.data
  to   = module.new_module.aws_s3_bucket.data
}
```

---

### 34. What is Terraform `check` block?
**Answer:**
`check` blocks (Terraform 1.5+) define assertions about infrastructure that run after `apply`. Unlike `precondition`/`postcondition`, they don't block the apply — they report warnings.

```hcl
check "health_check" {
  data "http" "app_health" {
    url = "https://${aws_lb.main.dns_name}/health"
  }

  assert {
    condition     = data.http.app_health.status_code == 200
    error_message = "Application health check failed with status ${data.http.app_health.status_code}"
  }
}

check "certificate_expiry" {
  data "tls_certificate" "app" {
    url = "https://${aws_lb.main.dns_name}"
  }

  assert {
    condition     = timecmp(data.tls_certificate.app.certificates[0].not_after, timeadd(timestamp(), "720h")) > 0
    error_message = "TLS certificate expires in less than 30 days!"
  }
}
```

---

### 35. What are Terraform Provisioners and when should you use them?
**Answer:**
Provisioners run scripts on a resource after it's created or before it's destroyed. They're considered a **last resort** — use them only when no native Terraform solution exists.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  key_name      = aws_key_pair.deploy.key_name

  # File provisioner — copy files to resource
  provisioner "file" {
    source      = "scripts/setup.sh"
    destination = "/tmp/setup.sh"

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  # Remote-exec provisioner — run commands on resource
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/setup.sh",
      "/tmp/setup.sh",
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  # Local-exec provisioner — run commands on machine running Terraform
  provisioner "local-exec" {
    command = "ansible-playbook -i '${self.public_ip},' playbook.yml"
  }

  # On-destroy provisioner
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance ${self.id} being destroyed' >> destroy.log"
  }
}
```

**Why avoid provisioners:**
- Not tracked in state
- Can fail silently
- Network-dependent
- Better alternatives: user_data, cloud-init, AWS Systems Manager, Ansible with dynamic inventory

---

### 36. What is `sensitive` in Terraform?
**Answer:**
The `sensitive` attribute marks values as sensitive — they won't appear in plan output or logs. Added to variables, outputs, and locals.

```hcl
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true   # Won't show in plan output
}

locals {
  connection_string = "postgresql://admin:${var.db_password}@${aws_db_instance.main.endpoint}/mydb"
  # This local is automatically sensitive because it references a sensitive value
}

output "db_connection_string" {
  value     = local.connection_string
  sensitive = true   # Required if output contains sensitive data
}
```

---

## ⚫ SECTION 7: SCENARIO-BASED QUESTIONS

---

### 37. SCENARIO: `terraform apply` failed halfway. Some resources were created, some weren't. What do you do?

**Answer:**
```bash
# Step 1: Don't panic. Terraform state is partially updated.
# Resources created before the failure ARE in state.

# Step 2: Read the error message carefully
# It tells you exactly what failed and why

# Step 3: Fix the issue (wrong AMI, insufficient permissions, quota exceeded)

# Step 4: Run terraform plan to see remaining changes
terraform plan

# Step 5: Apply again — Terraform will only create/update what's missing
terraform apply

# Terraform is idempotent — already-created resources won't be recreated
# It picks up from where it left off
```

---

### 38. SCENARIO: You need to rename a Terraform resource without destroying it. How?

**Answer:**
```hcl
# OLD config:
resource "aws_instance" "server" { ... }

# NEW config (renamed):
resource "aws_instance" "web_server" { ... }

# Terraform would DESTROY server and CREATE web_server!
# Fix: Add a moved block

moved {
  from = aws_instance.server
  to   = aws_instance.web_server
}
```

```bash
# Alternative: use terraform state mv
terraform state mv aws_instance.server aws_instance.web_server

# Then rename in .tf file and run plan — should show no changes
terraform plan   # Should show: No changes. Infrastructure is up-to-date.
```

---

### 39. SCENARIO: A resource was created manually (not via Terraform). How do you bring it under Terraform management?

**Answer:**
```hcl
# Step 1: Write the resource block in .tf file
resource "aws_s3_bucket" "logs" {
  bucket = "my-existing-logs-bucket"
}
```

```bash
# Step 2: Import the existing resource into Terraform state
terraform import aws_s3_bucket.logs my-existing-logs-bucket

# Step 3: Check plan for drift
terraform plan
# If there's drift, update .tf config to match reality

# Step 4: Apply to reconcile any differences
terraform apply
```

**Terraform 1.5+ config-driven import:**
```hcl
import {
  to = aws_s3_bucket.logs
  id = "my-existing-logs-bucket"
}

resource "aws_s3_bucket" "logs" {
  bucket = "my-existing-logs-bucket"
}
```

```bash
terraform plan    # Shows the import plan
terraform apply   # Imports and reconciles
```

---

### 40. SCENARIO: The Terraform state file is locked and no one is running a plan. How do you fix it?

**Answer:**
```bash
# Step 1: Verify no one is actually running Terraform
# Check CI/CD pipelines, ask teammates

# Step 2: Get the lock ID
terraform plan
# Error: Error acquiring the state lock
# Lock Info:
#   ID:        abc-123-xyz
#   Path:      s3://bucket/terraform.tfstate
#   Operation: OperationTypePlan
#   Who:       user@machine
#   Created:   2024-01-15 10:30:00 UTC

# Step 3: Force unlock (use only when you're certain no one is running)
terraform force-unlock abc-123-xyz

# Step 4: Check DynamoDB table for stale lock entry
aws dynamodb scan --table-name terraform-state-lock

# Step 5: Manually delete the lock item if needed
aws dynamodb delete-item \
  --table-name terraform-state-lock \
  --key '{"LockID": {"S": "s3://bucket/terraform.tfstate"}}'
```

---

### 41. SCENARIO: How do you upgrade Terraform provider versions safely?

**Answer:**
```bash
# Step 1: Check current versions
cat .terraform.lock.hcl

# Step 2: Update version constraint in required_providers
# ~> 4.0 → ~> 5.0

# Step 3: Check provider changelog for breaking changes
# https://github.com/hashicorp/terraform-provider-aws/blob/main/CHANGELOG.md

# Step 4: Update the lock file
terraform init -upgrade

# Step 5: Run plan to see what changes
terraform plan

# Step 6: Apply in non-production first
terraform apply

# Step 7: Monitor for issues, then promote to production

# Lock file best practices
git add .terraform.lock.hcl    # Always commit lock file!
git commit -m "chore: upgrade AWS provider to 5.x"
```

---

### 42. SCENARIO: How do you manage Terraform for a large team with multiple environments?

**Answer:**

**Recommended structure:**
```
infrastructure/
├── modules/                    # Shared, versioned modules
│   ├── vpc/
│   ├── eks/
│   └── rds/
├── environments/
│   ├── dev/
│   │   ├── backend.tf          # Separate S3 key
│   │   ├── main.tf             # Uses modules
│   │   ├── variables.tf
│   │   └── terraform.tfvars    # Dev-specific values
│   ├── staging/
│   └── production/
│       ├── backend.tf
│       ├── main.tf
│       └── terraform.tfvars
└── .github/workflows/
    └── terraform.yml           # CI/CD pipeline
```

```yaml
# GitHub Actions workflow
name: Terraform
on:
  pull_request:
    paths: ['infrastructure/**']
  push:
    branches: [main]
    paths: ['infrastructure/**']

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Terraform Init
        run: terraform init
        working-directory: infrastructure/environments/production

      - name: Terraform Plan (PR)
        if: github.event_name == 'pull_request'
        run: terraform plan -no-color
        working-directory: infrastructure/environments/production

      - name: Terraform Apply (main)
        if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve
        working-directory: infrastructure/environments/production
```

---

### 43. SCENARIO: How do you prevent accidental destruction of critical resources?

**Answer:**
```hcl
# Method 1: lifecycle prevent_destroy
resource "aws_db_instance" "production" {
  identifier = "prod-database"

  lifecycle {
    prevent_destroy = true   # Terraform errors if destroy is attempted
  }
}

# Method 2: Separate state for critical resources
# database/ has its own backend, separate from application/
# Even if application state is destroyed, database is untouched

# Method 3: IAM restrictions
# CI/CD role has no DeleteDB* permissions in production

# Method 4: Terraform Cloud — policy as code (Sentinel)
# policy "prevent-rds-destroy" {
#   rule = deny if plan.resource_changes contains
#          resource of type "aws_db_instance" with action "delete"
# }

# Method 5: Protected workspace in Terraform Cloud
# Mark workspace as protected — requires explicit override for destroy
```

---

### 44. SCENARIO: Your Terraform plan shows unexpected resource replacement (-/+). How do you investigate and prevent it?

**Answer:**
```bash
# Step 1: Read plan output carefully — why is it replacing?
# Look for "forces replacement" in plan details
# -/+ aws_instance.web (must be replaced)
#     ~ ami: "ami-old" → "ami-new" (forces replacement)

# Step 2: Common causes of replacement:
# - AMI changed (use lifecycle ignore_changes)
# - AZ changed
# - VPC/subnet changed
# - Name changed (some resources require name to be immutable)
# - Type changed (e.g., instance type for some resources)

# Step 3: Use lifecycle to prevent unintended replacements
```

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.latest.id    # Changes often
  instance_type = "t3.micro"

  lifecycle {
    ignore_changes = [ami]   # Don't replace instance when AMI changes
    # Update AMI via ASG rolling update instead
    create_before_destroy = true   # If replacement is needed, ensure no downtime
  }
}

# For resources where name forces replacement:
# Use a name prefix + random suffix
resource "aws_s3_bucket" "data" {
  bucket_prefix = "myapp-data-"   # Auto-generates unique name
  # vs bucket = "myapp-data"  (fixed name, can't change)
}
```

---

### 45. SCENARIO: How do you structure Terraform code to deploy a 3-tier web application?

**Answer:**

```hcl
# main.tf (root module)
module "networking" {
  source      = "./modules/networking"
  environment = var.environment
  vpc_cidr    = var.vpc_cidr
  azs         = var.availability_zones
}

module "security" {
  source  = "./modules/security"
  vpc_id  = module.networking.vpc_id
  environment = var.environment
}

module "database" {                         # Tier 3: Data layer
  source  = "./modules/rds"
  db_name = var.db_name
  db_user = var.db_username
  db_pass = var.db_password

  subnet_ids         = module.networking.private_subnet_ids
  security_group_ids = [module.security.db_sg_id]
  environment        = var.environment
}

module "application" {                      # Tier 2: App layer
  source     = "./modules/ecs"
  image      = "${var.ecr_repo}:${var.app_version}"
  db_host    = module.database.endpoint
  db_name    = var.db_name

  subnet_ids         = module.networking.private_subnet_ids
  security_group_ids = [module.security.app_sg_id]
  target_group_arn   = module.load_balancer.target_group_arn
  environment        = var.environment
}

module "load_balancer" {                    # Tier 1: Presentation layer
  source  = "./modules/alb"
  vpc_id  = module.networking.vpc_id

  subnet_ids         = module.networking.public_subnet_ids
  security_group_ids = [module.security.alb_sg_id]
  certificate_arn    = var.acm_certificate_arn
  environment        = var.environment
}

# outputs.tf
output "app_url" {
  value = "https://${module.load_balancer.dns_name}"
}
```

---

### 46. SCENARIO: How do you handle Terraform state when splitting a monolith into microservices?

**Answer:**

```bash
# Problem: Single huge state file with 500+ resources
# Solution: Split state into multiple smaller states

# Step 1: Plan the split
# networking.tfstate → VPC, subnets, route tables
# database.tfstate   → RDS, parameter groups
# app-service-a.tfstate → ECS, ALB for service A
# app-service-b.tfstate → ECS, ALB for service B
```

```hcl
# Step 2: Create new directory structure
# networking/
# database/
# services/service-a/
# services/service-b/
```

```bash
# Step 3: Move resources using state mv to new state files
terraform state mv \
  -state=old.tfstate \
  -state-out=networking/terraform.tfstate \
  aws_vpc.main aws_vpc.main

terraform state mv \
  -state=old.tfstate \
  -state-out=networking/terraform.tfstate \
  'aws_subnet.private[0]' 'aws_subnet.private[0]'

# Step 4: Update code references
# Use terraform_remote_state data source to share values between states
data "terraform_remote_state" "networking" {
  backend = "s3"
  config = {
    bucket = "tf-state"
    key    = "networking/terraform.tfstate"
    region = "us-east-1"
  }
}

# Step 5: Validate each new state
terraform plan   # Should show no changes
```

---

### 47. SCENARIO: How do you do a zero-downtime blue-green deployment with Terraform?

**Answer:**
```hcl
variable "active_color" {
  description = "Active deployment color (blue or green)"
  type        = string
  default     = "blue"
}

locals {
  colors   = ["blue", "green"]
  inactive = var.active_color == "blue" ? "green" : "blue"
}

# Deploy both blue and green ECS services
module "blue" {
  source = "./modules/ecs-service"
  color  = "blue"
  image  = var.blue_image
  count  = var.active_color == "blue" ? 1 : 0   # Only run active color
  target_group_arn = aws_lb_target_group.blue.arn
}

module "green" {
  source = "./modules/ecs-service"
  color  = "green"
  image  = var.green_image
  count  = var.active_color == "green" ? 1 : 0
  target_group_arn = aws_lb_target_group.green.arn
}

# ALB Listener rule — direct traffic to active color
resource "aws_lb_listener_rule" "main" {
  listener_arn = aws_lb_listener.https.arn
  action {
    type             = "forward"
    target_group_arn = var.active_color == "blue" ? aws_lb_target_group.blue.arn : aws_lb_target_group.green.arn
  }
  condition {
    path_pattern { values = ["/*"] }
  }
}

# Workflow:
# 1. Deploy green: terraform apply -var="active_color=blue" -var="green_image=myapp:v2"
# 2. Test green using test target group
# 3. Switch traffic: terraform apply -var="active_color=green"
# 4. Verify, then clean up blue
```

---

### 48. SCENARIO: You need to run the same Terraform code across 10 AWS accounts. How?

**Answer:**

```hcl
# Option 1: provider aliases with for_each
variable "aws_accounts" {
  default = {
    "dev"        = "111111111111"
    "staging"    = "222222222222"
    "production" = "333333333333"
  }
}

# Option 2: Separate backend config per account (recommended)
# accounts/dev/backend.tf
terraform {
  backend "s3" {
    bucket         = "tf-state-111111111111"
    key            = "global/terraform.tfstate"
    region         = "us-east-1"
    role_arn       = "arn:aws:iam::111111111111:role/TerraformRole"
  }
}

# Option 3: Terragrunt for DRY multi-account
# terragrunt.hcl (root)
locals {
  account_id = get_aws_account_id()
  region     = "us-east-1"
}

remote_state {
  backend = "s3"
  config = {
    bucket  = "tf-state-${local.account_id}"
    key     = "${path_relative_to_include()}/terraform.tfstate"
    region  = local.region
    encrypt = true
  }
}

# Run across all accounts with:
# terragrunt run-all plan
# terragrunt run-all apply
```

---

### 49. SCENARIO: Terraform apply is taking too long. How do you speed it up?

**Answer:**
```bash
# Problem 1: Too many resources in one state — split the state

# Problem 2: Terraform is refreshing state slowly
# Use -refresh=false if you trust the state is accurate
terraform plan -refresh=false
terraform apply -refresh=false

# Problem 3: Use -target to apply only specific resources
terraform apply -target=module.application -target=aws_ecs_service.web

# Problem 4: Use parallelism (default is 10 concurrent operations)
terraform apply -parallelism=20

# Problem 5: Provider API rate limits causing throttling
# Use -parallelism=5 for rate-limited providers

# Problem 6: Data source refreshing on every plan
# Use locals instead of data sources where possible

# Problem 7: Remote state access is slow
# Ensure backend is in the same region as where you're running Terraform

# Measurement
TF_LOG=TRACE terraform plan 2>&1 | grep "time="
```

---

### 50. SCENARIO: How do you manage database passwords and other secrets in Terraform?

**Answer:**

```hcl
# BAD — Never do this!
resource "aws_db_instance" "main" {
  password = "hardcoded-password-123"  # In .tf file or terraform.tfvars
}

# OPTION 1: Environment variables
# export TF_VAR_db_password=$(aws secretsmanager get-secret-value ...)
variable "db_password" {
  type      = string
  sensitive = true
}

resource "aws_db_instance" "main" {
  password = var.db_password
}

# OPTION 2: Read from AWS Secrets Manager at plan time
data "aws_secretsmanager_secret_version" "db_pass" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = jsondecode(data.aws_secretsmanager_secret_version.db_pass.secret_string)["password"]
}

# OPTION 3: Random password + store in Secrets Manager (best practice)
resource "random_password" "db" {
  length  = 32
  special = false
}

resource "aws_secretsmanager_secret" "db_pass" {
  name = "prod/rds/password"
}

resource "aws_secretsmanager_secret_version" "db_pass" {
  secret_id     = aws_secretsmanager_secret.db_pass.id
  secret_string = random_password.db.result
}

resource "aws_db_instance" "main" {
  password = random_password.db.result
}

# OPTION 4: HashiCorp Vault
data "vault_generic_secret" "db_creds" {
  path = "secret/prod/db"
}

resource "aws_db_instance" "main" {
  password = data.vault_generic_secret.db_creds.data["password"]
}
```

---

### 51. SCENARIO: How do you test Terraform code?

**Answer:**

**Level 1: Static Analysis**
```bash
terraform fmt -check         # Check formatting
terraform validate           # Validate syntax and logic
tflint                       # Lint for provider-specific issues
checkov -d .                 # Security and compliance scanning
tfsec .                      # Security scanning
infracost breakdown --path . # Cost estimation
```

**Level 2: Plan Testing**
```bash
# Always review plan output
terraform plan -detailed-exitcode
# Exit code 0 = no changes
# Exit code 1 = error
# Exit code 2 = changes present
```

**Level 3: Integration Testing with Terratest (Go)**
```go
// test/vpc_test.go
func TestVPC(t *testing.T) {
    opts := &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{
            "environment": "test",
            "cidr_block":  "10.0.0.0/16",
        },
    }

    defer terraform.Destroy(t, opts)
    terraform.InitAndApply(t, opts)

    vpcID := terraform.Output(t, opts, "vpc_id")
    assert.NotEmpty(t, vpcID)
}
```

**Level 4: Terraform native testing (1.6+)**
```hcl
# tests/vpc.tftest.hcl
run "vpc_is_created" {
  command = plan

  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block is incorrect"
  }
}
```

```bash
terraform test   # Run all tests
```

---

### 52. SCENARIO: How do you handle Terraform drift (infrastructure changed outside Terraform)?

**Answer:**
```bash
# Drift = real infrastructure ≠ Terraform state

# Step 1: Detect drift
terraform plan    # Shows what changed outside Terraform
# If Terraform wants to revert changes, that's drift

# Step 2: Refresh state to reflect reality (without changing infrastructure)
terraform refresh    # Updates state to match real infrastructure (deprecated in newer versions)
# Modern approach:
terraform apply -refresh-only    # Just refreshes, doesn't apply changes

# Step 3: Decide what to do
# Option A: Let Terraform fix drift (apply to revert manual changes)
terraform apply

# Option B: Accept the drift (update .tf files to match)
# Edit .tf files to reflect the manual changes, then:
terraform plan   # Should show no changes

# Step 4: Prevent future drift
# - Use IAM policies to restrict manual changes in production
# - Use AWS Config / Azure Policy to detect and alert on drift
# - Regular terraform plan runs in CI/CD to detect drift early
# - Use Terraform Cloud's drift detection feature
```

---

### 53. SCENARIO: How do you implement cost control with Terraform?

**Answer:**

```hcl
# 1. Use smaller instance types in non-prod (using variables)
variable "instance_type" {
  default = {
    dev        = "t3.micro"
    staging    = "t3.small"
    production = "t3.medium"
  }
}

resource "aws_instance" "web" {
  instance_type = var.instance_type[var.environment]
}

# 2. Auto-shutdown non-prod resources
resource "aws_autoscaling_schedule" "scale_down" {
  count                  = var.environment != "production" ? 1 : 0
  scheduled_action_name  = "scale-down-evenings"
  min_size               = 0
  max_size               = 0
  desired_capacity       = 0
  recurrence             = "0 20 * * 1-5"   # 8PM Mon-Fri
  autoscaling_group_name = aws_autoscaling_group.web.name
}

# 3. Tag everything for cost allocation
locals {
  mandatory_tags = {
    Environment  = var.environment
    Team         = var.team
    Project      = var.project_name
    CostCenter   = var.cost_center
    ManagedBy    = "Terraform"
  }
}

# 4. Use Infracost in CI/CD
# .github/workflows/infracost.yml
# - name: Run Infracost
#   run: infracost breakdown --path . --format json > infracost.json
# - name: Post cost comment on PR
#   run: infracost comment github --path infracost.json

# 5. S3 lifecycle policies to delete old objects
resource "aws_s3_bucket_lifecycle_configuration" "logs" {
  bucket = aws_s3_bucket.logs.id

  rule {
    id     = "expire-old-logs"
    status = "Enabled"
    expiration {
      days = 90
    }
  }
}
```

---

### 54. SCENARIO: How do you refactor Terraform code without causing downtime?

**Answer:**
```bash
# Scenario: Extracting a resource into a module

# BEFORE: inline resource
resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id
}

# AFTER: want to use a module
module "web_sg" {
  source = "./modules/security-group"
  name   = "web-sg"
  vpc_id = aws_vpc.main.id
}
```

```bash
# Step 1: Create the module and move resource definition

# Step 2: Tell Terraform about the rename (moved block)
```

```hcl
moved {
  from = aws_security_group.web
  to   = module.web_sg.aws_security_group.this
}
```

```bash
# Step 3: Verify plan shows no changes
terraform plan    # "No changes" - moved block worked

# Step 4: Apply (no resource recreation, just state update)
terraform apply

# Step 5: Remove moved block once merged to main
# (moved blocks can be cleaned up after the team pulls the latest)
```

---

### 55. SCENARIO: How do you share Terraform modules across teams?

**Answer:**

```hcl
# Option 1: Terraform Registry (public/private)
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"   # Public
  version = "~> 5.0"
}

module "internal" {
  source  = "app.terraform.io/myorg/vpc/aws"  # Private registry (TF Cloud)
  version = "~> 2.0"
}

# Option 2: Git repository with versioned tags
module "vpc" {
  source = "git::https://github.com/myorg/terraform-modules.git//aws/vpc?ref=v3.2.1"
}

# Option 3: S3/GCS bucket for internal distribution
module "vpc" {
  source = "s3::https://s3.amazonaws.com/my-tf-modules/vpc.zip"
}
```

**Module versioning best practices:**
```
terraform-modules/
├── aws/
│   ├── vpc/
│   │   ├── CHANGELOG.md
│   │   ├── README.md      # Always document inputs/outputs
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── eks/
└── .github/
    └── workflows/
        └── release.yml    # Auto-tag on main merge
```

```bash
# Semantic versioning for modules
git tag -a v2.1.0 -m "feat: add private endpoints support"
git push origin v2.1.0

# In consuming repo — pin to exact version in production
module "vpc" {
  source = "git::...//vpc?ref=v2.1.0"   # Never use main!
}
```

---

*© Terraform Interview Q&A — 55 Questions*
*Sections: Basics · State · Modules · Expressions · Workspaces · Advanced · 19 Scenarios*
