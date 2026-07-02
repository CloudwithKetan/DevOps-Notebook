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
