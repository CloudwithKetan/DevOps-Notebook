# AWS Services Interview Questions & Answers
> Covers EC2 · S3 · VPC · RDS · ECR · EKS · ECS · EBS · EFS · IAM · Route 53 · ASG · ELB

---

## ☁️ SECTION 1: EC2 (Elastic Compute Cloud)

---

### 1. What is Amazon EC2?
**Answer:**
Amazon EC2 (Elastic Compute Cloud) is a web service that provides resizable virtual servers (instances) in the cloud. It allows you to launch, configure, and manage virtual machines on demand.

**Key features:**
- Choose OS: Amazon Linux, Ubuntu, Windows, RHEL, etc.
- Multiple instance families (compute, memory, GPU, storage optimized)
- Pay only for what you use (per second billing for Linux)
- Elastic — scale up/down in minutes
- Full control via SSH/RDP

---

### 2. What are EC2 Instance Types?
**Answer:**

| Family | Purpose | Examples |
|---|---|---|
| **General Purpose** | Balanced compute/memory/network | t3, t4g, m5, m6i |
| **Compute Optimized** | High CPU workloads | c5, c6g, c7g |
| **Memory Optimized** | In-memory databases, big data | r5, r6i, x2idn |
| **Storage Optimized** | High I/O, NVMe storage | i3, i4i, d3 |
| **Accelerated Computing** | GPU, ML, HPC | p4, g5, inf2 |
| **HPC Optimized** | High Performance Computing | hpc6a |

**Instance naming:** `m5.xlarge`
- `m` = family (general purpose)
- `5` = generation
- `xlarge` = size (nano < micro < small < medium < large < xlarge < 2xlarge...)

---

### 3. What are EC2 Purchasing Options?
**Answer:**

| Option | Discount | Commitment | Use Case |
|---|---|---|---|
| **On-Demand** | No discount | None | Unpredictable, short-term |
| **Reserved (1yr/3yr)** | Up to 72% | 1 or 3 years | Steady-state workloads |
| **Savings Plans** | Up to 72% | 1 or 3 years | Flexible, cross-instance |
| **Spot Instances** | Up to 90% | None (can be interrupted) | Fault-tolerant batch jobs |
| **Dedicated Hosts** | Varies | On-demand or reserved | Licensing, compliance |
| **Dedicated Instances** | Premium | Per-instance | Isolation at hardware level |
| **Capacity Reservations** | None | None | Guaranteed capacity |

---

### 4. What is an AMI (Amazon Machine Image)?
**Answer:**
An AMI is a template that contains the software configuration (OS, application server, applications) required to launch an EC2 instance. It's like a snapshot of a configured server that can be used to launch multiple identical instances.

**AMI contains:**
- Root volume snapshot (OS, installed software)
- Launch permissions (who can use the AMI)
- Block device mapping (EBS volumes to attach)

```bash
# Create AMI from running instance
aws ec2 create-image \
  --instance-id i-1234567890abcdef0 \
  --name "MyApp-AMI-v1.0" \
  --no-reboot    # Don't stop instance during AMI creation

# Launch instance from AMI
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name my-key-pair
```

---

### 5. What is EC2 User Data?
**Answer:**
User Data is a script that runs automatically when an EC2 instance launches for the first time. Used for bootstrapping — installing packages, starting services, configuring the instance.

```bash
#!/bin/bash
# Example User Data script
yum update -y
yum install -y docker git
systemctl start docker
systemctl enable docker
usermod -a -G docker ec2-user

# Pull and run application
docker pull myregistry/myapp:latest
docker run -d -p 80:8080 myregistry/myapp:latest
```

- Runs as `root` user
- Runs only on **first boot** by default
- Max 16KB size
- Must start with `#!/bin/bash` (or other shebang)
- Viewable at: `http://169.254.169.254/latest/user-data`

---

### 6. What is EC2 Instance Metadata?
**Answer:**
Instance Metadata Service (IMDS) provides data about the running instance accessible from within the instance at `http://169.254.169.254/`.

```bash
# Get instance ID
curl http://169.254.169.254/latest/meta-data/instance-id

# Get instance type
curl http://169.254.169.254/latest/meta-data/instance-type

# Get public IP
curl http://169.254.169.254/latest/meta-data/public-ipv4

# Get IAM role credentials
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/MyRole

# IMDSv2 (more secure — requires session token)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

---

### 7. What are Security Groups and how do they work?
**Answer:**
Security Groups act as a virtual firewall for EC2 instances, controlling inbound and outbound traffic.

**Key characteristics:**
- **Stateful** — If inbound is allowed, the response is automatically allowed (no need for outbound rule)
- Default: Deny all inbound, Allow all outbound
- Rules are ALLOW only — no explicit deny
- Can reference other security groups (not just CIDR)
- Applied at the instance (ENI) level

```
Security Group: web-sg
Inbound:
  Port 80  (HTTP)  → 0.0.0.0/0     (from anywhere)
  Port 443 (HTTPS) → 0.0.0.0/0     (from anywhere)
  Port 22  (SSH)   → 10.0.0.0/8    (from VPN only)

Outbound:
  All traffic → 0.0.0.0/0           (to anywhere)
```

---

### 8. What is the difference between Security Groups and NACLs?
**Answer:**

| Feature | Security Group | Network ACL |
|---|---|---|
| Level | Instance (ENI) level | Subnet level |
| State | Stateful | Stateless (need both inbound + outbound rules) |
| Rules | Allow only | Allow AND Deny |
| Rule evaluation | All rules evaluated | Rules evaluated in number order |
| Default | Deny all inbound | Allow all inbound and outbound |
| Scope | Applies to specific instances | Applies to ALL instances in subnet |

**Best practice:** Use Security Groups as primary defense (instance level). Use NACLs as secondary defense (subnet level, e.g., block a known bad IP).

---

### 9. What is EC2 Key Pair?
**Answer:**
A key pair consists of a public key (stored by AWS) and a private key (downloaded by you). Used for SSH authentication to Linux instances.

```bash
# Create key pair
aws ec2 create-key-pair \
  --key-name my-keypair \
  --query 'KeyMaterial' \
  --output text > my-keypair.pem

chmod 400 my-keypair.pem

# Connect to instance
ssh -i my-keypair.pem ec2-user@<public-ip>

# For Windows instances — use key pair to decrypt admin password
```

---

### 10. What is EC2 Placement Groups?
**Answer:**
Placement Groups control how instances are placed on underlying hardware.

| Type | Description | Use Case |
|---|---|---|
| **Cluster** | All instances in same AZ, same rack — low latency | HPC, big data, ML training |
| **Spread** | Each instance on different hardware — max availability | Critical instances (max 7 per AZ) |
| **Partition** | Groups of instances on different partitions (different racks) | Large distributed systems (Kafka, Cassandra, HDFS) |

---

## 📦 SECTION 2: S3 (Simple Storage Service)

---

### 11. What is Amazon S3?
**Answer:**
Amazon S3 is an object storage service that provides unlimited, highly durable (11 9s = 99.999999999%), and highly available storage. Objects are stored in **buckets** and accessed via HTTP/HTTPS.

**Key concepts:**
- **Bucket** — Container for objects (globally unique name)
- **Object** — A file + metadata (max 5TB per object)
- **Key** — Object's unique identifier within a bucket (full path)
- **Region** — Bucket belongs to one region

```bash
# S3 URI: s3://bucket-name/path/to/file.txt
# HTTP URL: https://bucket-name.s3.amazonaws.com/path/to/file.txt

# CLI commands
aws s3 ls                           # List buckets
aws s3 ls s3://my-bucket/           # List objects in bucket
aws s3 cp file.txt s3://my-bucket/  # Upload file
aws s3 cp s3://my-bucket/file.txt . # Download file
aws s3 sync ./local-dir s3://my-bucket/prefix/  # Sync directory
aws s3 rm s3://my-bucket/file.txt   # Delete object
aws s3 mb s3://new-bucket           # Create bucket
aws s3 rb s3://my-bucket --force    # Delete bucket and contents
```

---

### 12. What are S3 Storage Classes?
**Answer:**

| Storage Class | Availability | Retrieval | Use Case |
|---|---|---|---|
| **Standard** | 99.99% | Instant | Frequently accessed data |
| **Standard-IA** | 99.9% | Instant | Infrequent access, minimum 30 days |
| **One Zone-IA** | 99.5% | Instant | Non-critical infrequent data (single AZ) |
| **Intelligent-Tiering** | 99.9% | Instant/Minutes/Hours | Unknown access patterns (auto-moves) |
| **Glacier Instant** | 99.9% | Instant | Archives accessed rarely |
| **Glacier Flexible** | 99.99% | Minutes to hours | Long-term archives |
| **Glacier Deep Archive** | 99.99% | 12 hours | 7-10 year retention (cheapest) |
| **Express One Zone** | 99.95% | Single-digit ms | High-performance, real-time |

---

### 13. What is S3 Versioning?
**Answer:**
S3 Versioning keeps multiple versions of an object. When you overwrite or delete an object, S3 creates a new version instead of modifying the original.

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# List all versions
aws s3api list-object-versions --bucket my-bucket

# Get specific version
aws s3api get-object \
  --bucket my-bucket \
  --key myfile.txt \
  --version-id "abc123" \
  myfile.txt

# Delete (creates a delete marker, not permanent deletion)
aws s3 rm s3://my-bucket/myfile.txt

# Permanently delete a version
aws s3api delete-object \
  --bucket my-bucket \
  --key myfile.txt \
  --version-id "abc123"
```

**MFA Delete** — Requires MFA authentication to permanently delete object versions. Extra protection against accidental/malicious deletion.

---

### 14. What is S3 Lifecycle Policy?
**Answer:**
Lifecycle policies automatically transition objects between storage classes or expire (delete) them after a specified time.

```json
{
  "Rules": [
    {
      "ID": "archive-and-delete",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [
        { "Days": 30, "StorageClass": "STANDARD_IA" },
        { "Days": 90, "StorageClass": "GLACIER" },
        { "Days": 365, "StorageClass": "DEEP_ARCHIVE" }
      ],
      "Expiration": { "Days": 2555 }
    }
  ]
}
```

---

### 15. What is S3 Bucket Policy vs S3 ACL?
**Answer:**

| Feature | Bucket Policy | ACL |
|---|---|---|
| Level | Bucket & object level | Object level |
| Format | JSON IAM policy | Predefined grants |
| Cross-account | Yes | Yes |
| Complexity | High (granular) | Simple (limited) |
| Recommended | Yes (modern approach) | Legacy (AWS recommends disabling) |

```json
// S3 Bucket Policy — allow public read for static website
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-website-bucket/*"
    }
  ]
}
```

---

### 16. What is S3 Cross-Region Replication (CRR) and Same-Region Replication (SRR)?
**Answer:**

**CRR** — Replicates objects to a bucket in a different AWS Region. Used for:
- Disaster recovery across regions
- Compliance (data sovereignty)
- Low latency access in different regions

**SRR** — Replicates to a bucket in the same region. Used for:
- Log aggregation
- Test environment data sync
- Maintain copies in same region

**Requirements for both:** Versioning must be enabled on source AND destination.

```bash
# Enable replication
aws s3api put-bucket-replication \
  --bucket source-bucket \
  --replication-configuration file://replication.json
```

---

### 17. What is S3 Encryption?
**Answer:**

**At Rest:**
| Type | Key Management | Description |
|---|---|---|
| **SSE-S3** | AWS managed | AWS manages keys (AES-256) |
| **SSE-KMS** | AWS KMS | Customer controls via KMS key |
| **SSE-C** | Customer provides | You provide key per request |
| **Client-side** | Customer manages entirely | Encrypt before upload |

**In Transit:** HTTPS/TLS always enabled. Enforce with bucket policy:
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": ["arn:aws:s3:::my-bucket/*"],
  "Condition": { "Bool": { "aws:SecureTransport": "false" } }
}
```

---

### 18. What is S3 Transfer Acceleration?
**Answer:**
S3 Transfer Acceleration speeds up uploads to S3 by routing traffic through AWS CloudFront's globally distributed edge locations. Your data is routed over the optimized AWS network backbone instead of the public internet.

**Enabled via:** Bucket Properties → Transfer Acceleration

**URL format:** `my-bucket.s3-accelerate.amazonaws.com`

Best for: Uploading large files from geographically distant locations.

---

## 🌐 SECTION 3: VPC (Virtual Private Cloud)

---

### 19. What is Amazon VPC?
**Answer:**
Amazon VPC lets you provision a logically isolated section of the AWS cloud where you launch AWS resources in a virtual network you define. You have complete control over:
- IP address ranges (CIDR blocks)
- Subnets (public and private)
- Route tables
- Network gateways
- Security settings (Security Groups, NACLs)

**Default VPC:** Every region has a default VPC (172.31.0.0/16) with public subnets in each AZ for quick instance launching.

---

### 20. What are Subnets in a VPC?
**Answer:**
Subnets are segments of a VPC's IP range located in a specific Availability Zone.

**Public Subnet:**
- Has a route to an Internet Gateway
- Resources can have public IPs
- For resources that need internet access (Load Balancers, Bastion hosts, NAT Gateways)

**Private Subnet:**
- No direct route to internet
- Instances can't be reached directly from the internet
- For resources that shouldn't be publicly accessible (databases, app servers)
- Use NAT Gateway for outbound internet access

**Subnet CIDR example for VPC 10.0.0.0/16:**
```
Public Subnets:
  10.0.1.0/24  → us-east-1a
  10.0.2.0/24  → us-east-1b
  10.0.3.0/24  → us-east-1c

Private Subnets:
  10.0.11.0/24 → us-east-1a
  10.0.12.0/24 → us-east-1b
  10.0.13.0/24 → us-east-1c
```

---

### 21. What is an Internet Gateway (IGW)?
**Answer:**
An Internet Gateway is a horizontally scaled, redundant, highly available VPC component that allows communication between instances in a VPC and the internet.

**Requirements for public subnet:**
1. Attach IGW to VPC
2. Add route `0.0.0.0/0 → igw-id` to subnet's route table
3. Instance must have a public or Elastic IP

**IGW does two things:**
- Allows instances with public IPs to communicate with internet
- Performs NAT for instances with public IPs

---

### 22. What is a NAT Gateway?
**Answer:**
A NAT (Network Address Translation) Gateway allows instances in **private subnets** to initiate outbound connections to the internet while preventing inbound connections.

```
Private Instance → NAT Gateway (in public subnet) → Internet Gateway → Internet
```

**Key facts:**
- NAT Gateway lives in a PUBLIC subnet
- Managed by AWS (no maintenance)
- Costs money per hour + per GB of data processed
- NOT highly available by itself — create one per AZ for HA
- Supports up to 45 Gbps bandwidth

```
Route table for private subnet:
  Destination   Target
  10.0.0.0/16   local
  0.0.0.0/0     nat-gateway-id   ← Send internet traffic to NAT Gateway
```

**NAT Gateway vs NAT Instance:**
| | NAT Gateway | NAT Instance |
|---|---|---|
| Managed by | AWS | You |
| Availability | AWS managed HA | Single point of failure |
| Bandwidth | Up to 45 Gbps | Depends on instance type |
| Cost | Higher | Lower |
| Maintenance | None | Patches, updates |

---

### 23. What is VPC Peering?
**Answer:**
VPC Peering is a networking connection between two VPCs that allows you to route traffic between them using private IP addresses. Works within a region, across regions, and across AWS accounts.

**Limitations:**
- No transitive peering (A↔B, B↔C does NOT mean A↔C)
- CIDR blocks must not overlap
- Need to update route tables on BOTH sides

```
Account A: VPC-1 (10.0.0.0/16)  ←peering→  VPC-2 (172.16.0.0/16) : Account B

Route table in VPC-1:
  172.16.0.0/16 → pcx-peering-id

Route table in VPC-2:
  10.0.0.0/16 → pcx-peering-id
```

---

### 24. What is AWS Transit Gateway?
**Answer:**
Transit Gateway is a network transit hub that connects VPCs and on-premises networks through a central hub. Solves the "mesh" problem of VPC peering (N*(N-1)/2 connections for N VPCs).

```
Without Transit Gateway (VPC Peering mesh):
  VPC-A ↔ VPC-B ↔ VPC-C ↔ VPC-D = 6 peering connections

With Transit Gateway:
  VPC-A → TGW ← VPC-B
  VPC-C → TGW ← VPC-D
  VPN   → TGW
  = 1 hub, supports transitive routing
```

**Supports:** VPCs, VPN connections, Direct Connect, other Transit Gateways (peering).

---

### 25. What is VPC Endpoint?
**Answer:**
A VPC Endpoint allows you to connect to AWS services privately without requiring internet gateway, NAT device, VPN, or Direct Connect. Traffic stays on the AWS network.

**Types:**
- **Interface Endpoint** — Elastic Network Interface with private IP. For most AWS services (SSM, SQS, SNS, Secrets Manager, etc.)
- **Gateway Endpoint** — Route table entry. Only for S3 and DynamoDB (free!)

```hcl
# Gateway endpoint for S3 (free)
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = aws_vpc.main.id
  service_name = "com.amazonaws.us-east-1.s3"
  route_table_ids = [aws_route_table.private.id]
}

# Interface endpoint for SSM
resource "aws_vpc_endpoint" "ssm" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ssm"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = [aws_subnet.private.id]
  security_group_ids  = [aws_security_group.endpoint.id]
  private_dns_enabled = true
}
```

---

## 🗄️ SECTION 4: RDS (Relational Database Service)

---

### 26. What is Amazon RDS?
**Answer:**
Amazon RDS is a managed relational database service that automates time-consuming administration tasks like hardware provisioning, database setup, patching, and backups.

**Supported engines:**
- Amazon Aurora (MySQL/PostgreSQL compatible — AWS proprietary, fastest)
- MySQL
- PostgreSQL
- MariaDB
- Oracle
- Microsoft SQL Server

**What AWS manages:**
- Hardware provisioning
- OS patching
- Database patching
- Automated backups
- Monitoring and alerting
- Multi-AZ failover

---

### 27. What is RDS Multi-AZ?
**Answer:**
Multi-AZ deploys a synchronous standby replica in a different Availability Zone. If the primary fails, RDS automatically fails over to the standby within 1-2 minutes.

```
Primary DB (us-east-1a) ──sync replication──→ Standby DB (us-east-1b)
           ↑
     Single DNS endpoint (unchanged during failover)
```

**Key points:**
- Standby is NOT accessible for reads (only for failover)
- Automatic failover — no manual intervention needed
- DNS endpoint doesn't change (application reconnects automatically)
- For read performance, use **Read Replicas** (not Multi-AZ)
- Backups taken from standby (no I/O impact on primary)

---

### 28. What are RDS Read Replicas?
**Answer:**
Read Replicas use **asynchronous** replication to create one or more copies of your database for read-heavy workloads. They serve read traffic, reducing load on the primary.

```
Primary DB (writes + reads) ──async replication──→ Read Replica 1 (reads only)
                              ──async replication──→ Read Replica 2 (reads only)
```

**Key points:**
- Up to 15 read replicas for Aurora, 5 for other engines
- Can be in same AZ, different AZ, or different Region (for DR)
- Has its own DNS endpoint (application must be configured to use it)
- Can be promoted to standalone DB (for migrations)
- Small replication lag (asynchronous)

**Multi-AZ vs Read Replica:**
| | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | High Availability | Read Scaling |
| Replication | Synchronous | Asynchronous |
| Readable? | No (standby only) | Yes |
| Region? | Same region | Same or different |
| Failover? | Automatic | Manual promotion |

---

### 29. What is Amazon Aurora?
**Answer:**
Aurora is AWS's proprietary cloud-optimized relational database compatible with MySQL and PostgreSQL. It provides up to 5x performance over MySQL and 3x over PostgreSQL.

**Aurora architecture:**
- Distributed storage (6 copies across 3 AZs)
- Auto-heals storage
- Supports up to 15 low-latency read replicas
- Continuous backup to S3
- Fast recovery (< 30 seconds)

**Aurora Serverless v2:**
- Auto-scales capacity in fine-grained increments
- Pay per ACU (Aurora Capacity Unit) second
- For intermittent, unpredictable workloads

---

### 30. What is RDS Automated Backup and Snapshot?
**Answer:**

| | Automated Backup | Manual Snapshot |
|---|---|---|
| Triggered | Automatically (daily) | Manually or via API |
| Retention | 1–35 days | Until you delete |
| Type | Full + transaction logs (PITR) | Full backup |
| Performance impact | None (taken from standby) | Brief I/O suspension |
| After DB delete | Deleted | Retained |

**Point-in-Time Recovery (PITR):**
- Restore to any second within the backup retention period
- Uses automated backups + transaction logs

```bash
# Restore to point in time
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier mydb \
  --target-db-instance-identifier mydb-restored \
  --restore-time 2024-01-15T10:30:00Z
```

---

## 📦 SECTION 5: ECR (Elastic Container Registry)

---

### 31. What is Amazon ECR?
**Answer:**
Amazon ECR is a fully managed Docker container registry that makes it easy to store, manage, and deploy Docker container images. It integrates seamlessly with ECS, EKS, and Lambda.

**Key features:**
- Private and public registries
- Image scanning (vulnerability detection)
- Lifecycle policies (auto-delete old images)
- Cross-region and cross-account replication
- Immutable tags (prevent tag overwriting)
- Integrated with IAM for access control

```bash
# Authenticate
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# Create repository
aws ecr create-repository --repository-name my-app

# Build, tag, and push
docker build -t my-app .
docker tag my-app:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.0
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.0

# Pull image
docker pull 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.0
```

---

### 32. What is ECR Image Scanning?
**Answer:**
ECR can automatically scan container images for software vulnerabilities using Common Vulnerabilities and Exposures (CVEs).

**Scanning types:**
- **Basic Scanning** — Uses Clair for OS package vulnerabilities (free)
- **Enhanced Scanning** — Uses Amazon Inspector for deeper scanning (OS + programming languages)

```bash
# Trigger manual scan
aws ecr start-image-scan \
  --repository-name my-app \
  --image-id imageTag=v1.0

# Get scan results
aws ecr describe-image-scan-findings \
  --repository-name my-app \
  --image-id imageTag=v1.0
```

---

### 33. What is ECR Lifecycle Policy?
**Answer:**
ECR Lifecycle policies automatically clean up unused images to reduce storage costs.

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep only 10 tagged releases",
      "selection": {
        "tagStatus": "tagged",
        "tagPrefixList": ["v"],
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": { "type": "expire" }
    },
    {
      "rulePriority": 2,
      "description": "Delete untagged images older than 7 days",
      "selection": {
        "tagStatus": "untagged",
        "countType": "sinceImagePushed",
        "countUnit": "days",
        "countNumber": 7
      },
      "action": { "type": "expire" }
    }
  ]
}
```

---

## ☸️ SECTION 6: EKS (Elastic Kubernetes Service)

---

### 34. What is Amazon EKS?
**Answer:**
Amazon EKS is a fully managed Kubernetes service. AWS manages the Kubernetes control plane (API server, etcd, scheduler, controllers), and you manage the worker nodes (or use Fargate for serverless nodes).

**AWS manages:**
- Control plane (HA across 3 AZs)
- etcd
- Kubernetes updates and patches
- Control plane security

**You manage:**
- Worker nodes (EC2 or Fargate)
- Node group updates
- Application deployments
- Networking (VPC CNI)
- Storage (EBS/EFS CSI drivers)

---

### 35. What are EKS Node Types?
**Answer:**

| Node Type | Description | Use Case |
|---|---|---|
| **Managed Node Groups** | AWS manages EC2 lifecycle, AMI updates | Standard workloads |
| **Self-managed Nodes** | You fully control the EC2 instances | Custom AMIs, special configs |
| **Fargate Nodes** | Serverless — AWS manages all nodes | No node management, unpredictable scale |

```bash
# Create managed node group
aws eks create-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name workers \
  --node-role arn:aws:iam::123456789:role/EKSNodeRole \
  --scaling-config minSize=2,maxSize=10,desiredSize=3 \
  --instance-types t3.medium \
  --subnets subnet-abc subnet-def
```

---

### 36. What is the EKS Add-ons?
**Answer:**
EKS Add-ons are managed software components that AWS maintains and updates automatically.

| Add-on | Purpose |
|---|---|
| **VPC CNI** | Pod networking using VPC IPs |
| **CoreDNS** | DNS service for pods |
| **kube-proxy** | Network rules on each node |
| **EBS CSI Driver** | Dynamic EBS volume provisioning |
| **EFS CSI Driver** | EFS volume mounting |
| **AWS Load Balancer Controller** | ALB/NLB from Kubernetes Ingress/Service |
| **Cluster Autoscaler** | Auto-scale node groups |
| **ADOT** | AWS Distro for OpenTelemetry (monitoring) |

```bash
# List available add-ons
aws eks describe-addon-versions --kubernetes-version 1.28

# Install add-on
aws eks create-addon \
  --cluster-name my-cluster \
  --addon-name aws-ebs-csi-driver \
  --addon-version v1.24.0-eksbuild.1
```

---

### 37. What is IRSA (IAM Roles for Service Accounts)?
**Answer:**
IRSA allows Kubernetes pods to assume AWS IAM roles by associating an IAM role with a Kubernetes Service Account. The pod gets temporary credentials via OIDC federation — no long-lived AWS credentials needed.

```bash
# Enable OIDC provider
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster --approve

# Create IAM role for service account
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --namespace production \
  --name s3-access-sa \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve
```

```yaml
# Use service account in pod
spec:
  serviceAccountName: s3-access-sa
  containers:
  - name: app
    image: myapp:latest
    # Pod can now access S3 without any AWS credentials!
```

---

## 🐳 SECTION 7: ECS (Elastic Container Service)

---

### 38. What is Amazon ECS?
**Answer:**
Amazon ECS is a fully managed container orchestration service. Unlike EKS (Kubernetes), ECS is AWS's proprietary orchestration platform — simpler to set up but less portable.

**ECS components:**
| Component | Description |
|---|---|
| **Cluster** | Logical grouping of compute resources |
| **Task Definition** | Blueprint for containers (like docker-compose) |
| **Task** | Running instance of a task definition |
| **Service** | Maintains desired count of tasks, handles scaling + LB |
| **Container Agent** | Runs on each EC2 instance, communicates with ECS |

---

### 39. What is the difference between ECS EC2 and ECS Fargate?
**Answer:**

| Feature | ECS EC2 | ECS Fargate |
|---|---|---|
| Infrastructure | You manage EC2 instances | AWS manages all infrastructure |
| Scaling | Manage cluster capacity + task count | Only manage task count |
| Pricing | Pay for EC2 instances | Pay per vCPU/memory per second |
| Control | Full control over nodes | No node access |
| Use case | Cost-optimized, GPU, custom | Serverless, simplicity |
| Startup time | Fast | Slightly slower (cold start) |

---

### 40. What is an ECS Task Definition?
**Answer:**
A Task Definition is a JSON blueprint that describes one or more containers. It specifies image, CPU, memory, networking, volumes, environment variables, logging, etc.

```json
{
  "family": "my-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::123:role/ECSExecutionRole",
  "taskRoleArn": "arn:aws:iam::123:role/ECSTaskRole",
  "containerDefinitions": [
    {
      "name": "my-app",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.0",
      "portMappings": [{ "containerPort": 8080 }],
      "environment": [
        { "name": "NODE_ENV", "value": "production" }
      ],
      "secrets": [
        { "name": "DB_PASSWORD", "valueFrom": "arn:aws:secretsmanager:..." }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3
      }
    }
  ]
}
```

---

## 💾 SECTION 8: EBS (Elastic Block Store)

---

### 41. What is Amazon EBS?
**Answer:**
EBS provides persistent block storage volumes for EC2 instances. Like a hard drive attached to your server — data persists independently of the instance lifecycle.

**Key characteristics:**
- Attached to ONE instance at a time (except Multi-Attach for io2)
- Stays in ONE Availability Zone
- Automatically replicated within the AZ
- Snapshots backed up to S3 (cross-AZ/region capable)
- Can be encrypted with KMS

---

### 42. What are EBS Volume Types?
**Answer:**

| Type | Category | Max IOPS | Max Throughput | Use Case |
|---|---|---|---|---|
| **gp3** | SSD | 16,000 | 1,000 MB/s | General purpose (default, cheapest SSD) |
| **gp2** | SSD | 16,000 | 250 MB/s | Legacy general purpose |
| **io2 Block Express** | SSD | 256,000 | 4,000 MB/s | Critical databases (Oracle, SAP) |
| **io1** | SSD | 64,000 | 1,000 MB/s | Databases needing high IOPS |
| **st1** | HDD | 500 | 500 MB/s | Big data, data warehouses |
| **sc1** | HDD | 250 | 250 MB/s | Cold storage (cheapest) |

**gp3 vs gp2:**
- gp3 is cheaper and allows independent IOPS/throughput scaling
- gp2 IOPS is tied to size (3 IOPS/GB, max 16,000)
- Always prefer gp3 for new volumes

---

### 43. What is an EBS Snapshot?
**Answer:**
EBS Snapshots are point-in-time backups of EBS volumes stored in S3. Snapshots are incremental — only changed blocks since the last snapshot are saved.

```bash
# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-1234567890abcdef0 \
  --description "Daily backup"

# List snapshots
aws ec2 describe-snapshots --owner-ids self

# Create volume from snapshot (can be in different AZ!)
aws ec2 create-volume \
  --snapshot-id snap-1234567890abcdef0 \
  --availability-zone us-east-1b \
  --volume-type gp3

# Copy snapshot to another region (for DR)
aws ec2 copy-snapshot \
  --source-region us-east-1 \
  --source-snapshot-id snap-abc123 \
  --destination-region eu-west-1
```

---

### 44. What is EBS Multi-Attach?
**Answer:**
EBS Multi-Attach (available for io1/io2 volumes) allows a single EBS volume to be attached to multiple EC2 instances simultaneously within the same AZ.

**Requirements:**
- Only io1/io2 volume types
- Instances must be in the same AZ
- Linux only (must use cluster-aware filesystem like GFS2, OCFS2)
- Not for general use — specific use cases only

**Use cases:** High-availability applications, clustered databases, shared storage.

---

## 📂 SECTION 9: EFS (Elastic File System)

---

### 45. What is Amazon EFS?
**Answer:**
EFS is a fully managed, scalable, elastic NFS (Network File System) that can be shared across multiple EC2 instances and even Lambda functions simultaneously.

**Key characteristics:**
- **Shared storage** — Multiple instances mount the same filesystem
- **Elastic** — Automatically grows and shrinks (no provisioning needed)
- **Multi-AZ** — Stored redundantly across multiple AZs
- **Linux only** — NFSv4 protocol (not for Windows)
- **Pay per use** — No minimum commitment

---

### 46. What is the difference between EBS and EFS?
**Answer:**

| Feature | EBS | EFS |
|---|---|---|
| Type | Block storage | File storage (NFS) |
| Access | 1 instance (except Multi-Attach) | Thousands of instances simultaneously |
| AZ scope | Single AZ | Multi-AZ (regional) |
| Scaling | Fixed size (can resize) | Automatic elastic scaling |
| Protocol | - | NFSv4 |
| OS support | Linux + Windows | Linux only |
| Performance | Faster, lower latency | Higher latency (network) |
| Price | Cheaper per GB | More expensive per GB |
| Use case | OS disk, databases | Shared content, CMS, home directories |

---

### 47. What are EFS Storage Classes?
**Answer:**

| Class | Access Pattern | Cost |
|---|---|---|
| **Standard** | Frequently accessed | Higher |
| **Standard-IA (Infrequent Access)** | Infrequent access | Lower storage, retrieval fee |
| **One Zone** | Frequently accessed, single AZ | 47% cheaper than Standard |
| **One Zone-IA** | Infrequent, single AZ | Cheapest |

**EFS Lifecycle Management:** Automatically moves files to IA storage after N days of no access (7, 14, 30, 60, 90 days).

---

## 🔐 SECTION 10: IAM (Identity and Access Management)

---

### 48. What is AWS IAM?
**Answer:**
IAM is a web service that helps you securely control access to AWS resources. It manages **authentication** (who you are) and **authorization** (what you can do).

**IAM components:**
| Component | Description |
|---|---|
| **Users** | Individual people or applications |
| **Groups** | Collection of users with shared permissions |
| **Roles** | Assumed by AWS services, users, or external identities |
| **Policies** | JSON documents defining permissions |
| **Identity Providers** | SAML, OIDC for federation |

**Root account:** Full access, should NEVER be used for daily tasks. Enable MFA, create admin user instead.

---

### 49. What is an IAM Policy?
**Answer:**
IAM Policies are JSON documents that define permissions — what actions are allowed or denied on what resources.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    },
    {
      "Sid": "DenyDeleteS3",
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**Policy types:**
| Type | Managed By | Reusable |
|---|---|---|
| **AWS Managed** | AWS | Yes (can't edit) |
| **Customer Managed** | You | Yes |
| **Inline** | Embedded in identity | No (1:1 relationship) |
| **Resource-based** | Attached to resource | (S3 bucket policy, KMS) |
| **SCP (Service Control Policy)** | AWS Organizations | Org-wide guardrails |

---

### 50. What is an IAM Role?
**Answer:**
An IAM Role is an identity with specific permissions that can be assumed by AWS services, users, or external identities. Unlike users, roles don't have long-term credentials — they provide temporary credentials via STS.

```json
// Trust Policy — who can assume this role
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "ec2.amazonaws.com"   // EC2 can assume this role
    },
    "Action": "sts:AssumeRole"
  }]
}
```

**Common role use cases:**
- EC2 instance role — instance accesses S3, SSM, CloudWatch
- Lambda execution role — function accesses DynamoDB, SQS
- ECS task role — container accesses secrets, S3
- Cross-account role — Account A assumes role in Account B
- CI/CD role — GitHub Actions assumes role via OIDC

---

### 51. What is IAM Permission Boundary?
**Answer:**
A Permission Boundary is an advanced IAM feature that sets the maximum permissions an IAM principal can have. Even if you attach a full admin policy, the boundary limits what's actually allowed.

```
Effective Permissions = Identity Policy ∩ Permission Boundary
```

```json
// Permission Boundary — limits user to S3 and EC2 only
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:*", "ec2:*"],
    "Resource": "*"
  }]
}
// Even if this user has AdministratorAccess policy,
// they can ONLY do S3 and EC2 operations
```

**Use case:** Allow developers to create IAM roles for their applications but prevent privilege escalation.

---

## 🌍 SECTION 11: ROUTE 53

---

### 52. What is Amazon Route 53?
**Answer:**
Amazon Route 53 is a highly available, scalable cloud DNS (Domain Name System) web service. It connects user requests to infrastructure running in AWS and outside.

**Three main functions:**
1. **Domain registration** — Register and manage domain names
2. **DNS routing** — Translates domain names to IP addresses
3. **Health checking** — Monitor endpoint health and route accordingly

---

### 53. What are Route 53 Record Types?
**Answer:**

| Record | Purpose | Example |
|---|---|---|
| **A** | Maps hostname to IPv4 | `api.example.com → 1.2.3.4` |
| **AAAA** | Maps hostname to IPv6 | `api.example.com → ::1` |
| **CNAME** | Maps hostname to another hostname | `www → api.example.com` |
| **Alias** | AWS-specific, maps to AWS resource | `app.example.com → my-alb.amazonaws.com` |
| **MX** | Mail exchange servers | For email routing |
| **TXT** | Text records | SPF, DKIM, domain verification |
| **NS** | Name server records | Delegates domain to Route 53 |
| **SOA** | Start of Authority | Zone metadata |
| **PTR** | Reverse DNS lookup | IP to hostname |
| **SRV** | Service location | For specific protocols |

**CNAME vs Alias:**
- CNAME can't be used at zone apex (`example.com`)
- Alias can be used at zone apex and is free (CNAME charges for queries)
- Alias only points to AWS resources (ALB, CloudFront, S3, etc.)

---

### 54. What are Route 53 Routing Policies?
**Answer:**

| Policy | Description | Use Case |
|---|---|---|
| **Simple** | Single record, no health checks | Basic routing to one resource |
| **Weighted** | % split between multiple resources | A/B testing, gradual migration |
| **Latency** | Routes to lowest latency region | Global applications |
| **Failover** | Primary/standby with health check | Disaster recovery |
| **Geolocation** | Routes based on user's geography | Content localization, compliance |
| **Geoproximity** | Routes based on geography + bias | Fine-grained geographic control |
| **Multivalue Answer** | Returns multiple healthy IPs | Simple load balancing (not a replacement for ELB) |
| **IP-based** | Routes based on client IP CIDR | Route ISP/corporate users differently |

```bash
# Weighted routing example: 90% to v1, 10% to v2
aws route53 change-resource-record-sets \
  --hosted-zone-id ZONE_ID \
  --change-batch '{
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "api.example.com",
          "Type": "A",
          "SetIdentifier": "v1",
          "Weight": 90,
          "TTL": 60,
          "ResourceRecords": [{"Value": "1.2.3.4"}]
        }
      }
    ]
  }'
```

---

## 📈 SECTION 12: ASG (Auto Scaling Group)

---

### 55. What is an Auto Scaling Group?
**Answer:**
An ASG automatically adjusts the number of EC2 instances based on demand. It ensures you have the right number of instances available to handle your application load.

**ASG components:**
- **Launch Template/Configuration** — What to launch (AMI, instance type, SG, user data)
- **Min/Max/Desired capacity** — Scaling boundaries
- **Scaling policies** — When to scale
- **Health checks** — EC2 or ELB health checks
- **AZs** — Which AZs to launch instances in

---

### 56. What are ASG Scaling Policies?
**Answer:**

**1. Target Tracking Scaling (recommended)**
```
Maintain CPU at 50% → ASG adds/removes instances automatically
```
```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0
  }'
```

**2. Step Scaling**
```
CPU > 70% → add 2 instances
CPU > 90% → add 4 instances
CPU < 30% → remove 1 instance
```

**3. Simple Scaling**
```
CloudWatch alarm triggers → add/remove N instances, then wait (cooldown)
```

**4. Scheduled Scaling**
```
Every weekday at 8 AM → set desired to 10
Every weekday at 8 PM → set desired to 2
```

**5. Predictive Scaling**
- ML-based, predicts future load and proactively scales

---

### 57. What is the ASG Cooldown Period?
**Answer:**
The cooldown period (default 300 seconds) is a wait period after a scaling activity before ASG can initiate another scaling action. Prevents rapid "thrashing" of instances.

**Cooldown vs Warm-up:**
- **Cooldown** — Wait period AFTER scaling activity
- **Warm-up** — Time NEW instance takes to start serving traffic (excluded from scaling metrics during this time)

```bash
# Set cooldown
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --default-cooldown 120    # 2 minutes
```

---

### 58. What is ASG Instance Refresh?
**Answer:**
Instance Refresh allows you to update instances in the ASG with a new Launch Template (new AMI, new configuration) while maintaining availability.

```bash
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name my-asg \
  --preferences '{
    "MinHealthyPercentage": 90,       # Keep 90% healthy during refresh
    "InstanceWarmup": 60,
    "CheckpointPercentages": [20, 50, 100],
    "CheckpointDelay": 600
  }'
```

---

## ⚖️ SECTION 13: ELB (Elastic Load Balancing)

---

### 59. What is Elastic Load Balancing?
**Answer:**
ELB automatically distributes incoming application traffic across multiple targets (EC2 instances, containers, Lambda, IP addresses) in one or more Availability Zones.

**Three types of load balancers:**

| Type | Layer | Protocol | Use Case |
|---|---|---|---|
| **ALB (Application)** | Layer 7 (HTTP) | HTTP, HTTPS, WebSocket | Web apps, microservices, containers |
| **NLB (Network)** | Layer 4 (TCP/UDP) | TCP, UDP, TLS | High performance, static IP, gaming |
| **GWLB (Gateway)** | Layer 3 (IP) | IP | Firewall appliances, IDS/IPS |
| **CLB (Classic)** | Layer 4/7 | HTTP, HTTPS, TCP | Legacy (deprecated) |

---

### 60. What is ALB and its key features?
**Answer:**
Application Load Balancer operates at Layer 7 (HTTP/HTTPS) and supports advanced routing.

**Key features:**
- **Path-based routing** — `/api/*` → API service, `/static/*` → S3
- **Host-based routing** — `api.example.com` → API servers, `web.example.com` → Web servers
- **HTTP header/method routing** — Route based on headers, query strings, source IP
- **Target groups** — Group of targets (instances, IPs, Lambda, containers)
- **Weighted target groups** — Canary deployments (90%/10% split)
- **SSL/TLS termination** — Offload HTTPS to ALB, HTTP to backends
- **WebSocket support** — Long-lived connections
- **HTTP/2** — Multiplexed requests
- **WAF integration** — Web Application Firewall

```bash
# ALB rule: route /api/* to api-tg, /* to web-tg
aws elbv2 create-rule \
  --listener-arn arn:aws:... \
  --conditions '[{"Field":"path-pattern","Values":["/api/*"]}]' \
  --actions '[{"Type":"forward","TargetGroupArn":"arn:aws:...api-tg"}]' \
  --priority 10
```

---

### 61. What is NLB and when do you use it?
**Answer:**
Network Load Balancer operates at Layer 4 (TCP/UDP) and is designed for extreme performance.

**Use NLB when:**
- Need millions of requests per second
- Ultra-low latency (microseconds)
- Need static IP address (NLB has static Elastic IPs)
- Non-HTTP protocols (TCP, UDP, TLS)
- Gaming, IoT, financial trading applications

**NLB key features:**
- Static IP per AZ (great for whitelisting)
- Preserves source IP (unlike ALB)
- No SSL termination at application level
- Supports PrivateLink

---

### 62. What is a Target Group?
**Answer:**
A Target Group is a collection of targets (instances, IPs, Lambda functions) that receive traffic from a load balancer. Health checks are configured at the target group level.

```bash
# Create target group
aws elbv2 create-target-group \
  --name my-app-tg \
  --protocol HTTP \
  --port 8080 \
  --vpc-id vpc-abc123 \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --target-type instance    # or ip, lambda

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:... \
  --targets Id=i-abc123 Id=i-def456
```

---

### 63. What is ALB Sticky Sessions?
**Answer:**
Sticky Sessions (session affinity) route requests from the same client to the same target, useful for stateful applications.

- **Duration-based stickiness** — ALB generates a cookie, expires after N seconds
- **Application-based stickiness** — Application generates cookie, ALB respects it

**Downside:** Uneven load distribution. Prefer stateless applications instead.

---

## 🔥 SECTION 14: SCENARIO-BASED QUESTIONS

---

### 64. SCENARIO: EC2 instance is unreachable via SSH. How do you troubleshoot?

**Answer:**
```bash
# Step 1: Check instance state
aws ec2 describe-instances --instance-ids i-abc123 | grep State

# Step 2: Check Security Group inbound rules
# Must allow port 22 from your IP or CIDR

# Step 3: Check NACL — ensure port 22 inbound AND ephemeral ports outbound allowed

# Step 4: Check if instance has a public IP (for public subnet)
aws ec2 describe-instances --instance-ids i-abc123 | grep PublicIpAddress

# Step 5: Check route table — public subnet must have 0.0.0.0/0 → IGW

# Step 6: Check if SSH service is running (use EC2 Serial Console or SSM)
# AWS Console → EC2 → Connect → EC2 Serial Console (no SSH needed)
# or
aws ssm start-session --target i-abc123  # SSM Session Manager (no SSH needed)

# Step 7: Check System Log for boot errors
aws ec2 get-console-output --instance-id i-abc123

# Step 8: Verify key pair is correct
# If wrong key pair — detach volume, attach to another instance, fix authorized_keys

# Common fixes:
# - Add port 22 to security group
# - Assign Elastic IP if no public IP
# - Use SSM instead of SSH (best practice — no open SSH port needed)
```

---

### 65. SCENARIO: S3 bucket objects are accidentally deleted. How do you recover?

**Answer:**
```bash
# If versioning was ENABLED:
# 1. List delete markers
aws s3api list-object-versions \
  --bucket my-bucket \
  --query 'DeleteMarkers[?Key==`myfile.txt`]'

# 2. Delete the delete marker to restore the object
aws s3api delete-object \
  --bucket my-bucket \
  --key myfile.txt \
  --version-id "delete-marker-version-id"

# 3. Object is now restored to its previous version

# If versioning was NOT enabled:
# Objects are permanently deleted — no recovery via S3

# Recovery options without versioning:
# - Restore from S3 backup (CRR destination bucket)
# - Check CloudTrail for who deleted (forensics)
# - Check if data exists in EBS/RDS snapshots

# Prevention:
# 1. Enable versioning IMMEDIATELY
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# 2. Enable MFA Delete for critical buckets
# 3. Use S3 Object Lock for compliance (WORM - Write Once Read Many)
# 4. Use bucket policy to deny DeleteObject
```

---

### 66. SCENARIO: RDS database is running slow. How do you diagnose and fix it?

**Answer:**
```bash
# Step 1: Check CloudWatch metrics
# - CPUUtilization > 80%?
# - DatabaseConnections at limit?
# - ReadIOPS/WriteIOPS throttled?
# - FreeableMemory critically low?

# Step 2: Enable and check Performance Insights
# AWS Console → RDS → Performance Insights
# Shows: Top SQL, wait events, top hosts

# Step 3: Check for long-running queries
# Connect to DB and run:
SELECT * FROM information_schema.processlist
WHERE time > 30 ORDER BY time DESC;

# Step 4: Check slow query log
# Enable: set global slow_query_log = 'ON';
# set global long_query_time = 1;  -- Log queries > 1 second

# Solutions based on findings:
# CPU high → Add read replicas, optimize queries, add indexes
# Connections exhausted → Use RDS Proxy (connection pooling)
# Storage I/O throttled → Upgrade to gp3, increase IOPS, upgrade instance
# Memory low → Upgrade instance class
# Slow queries → Add database indexes, optimize queries, use caching (ElastiCache)
```

---

### 67. SCENARIO: Your application needs to read from S3 but is running on EC2. How do you set up access securely?

**Answer:**
```bash
# WRONG approach — hardcode credentials
# export AWS_ACCESS_KEY_ID=AKIA...  ← NEVER DO THIS

# CORRECT approach — EC2 Instance Profile
```

```json
// Step 1: Create IAM policy
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": [
      "arn:aws:s3:::my-bucket",
      "arn:aws:s3:::my-bucket/*"
    ]
  }]
}
```

```bash
# Step 2: Create IAM role with EC2 trust policy
aws iam create-role \
  --role-name EC2-S3-ReadOnly \
  --assume-role-policy-document file://ec2-trust.json

# Step 3: Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-ReadOnly \
  --policy-arn arn:aws:iam::123:policy/S3ReadPolicy

# Step 4: Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-ReadOnly

# Step 5: Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-ReadOnly \
  --role-name EC2-S3-ReadOnly

# Step 6: Attach to EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-abc123 \
  --iam-instance-profile Name=EC2-S3-ReadOnly

# Now the EC2 instance can access S3 using temporary credentials from IMDS
# aws s3 ls s3://my-bucket — works without any credentials configured!
```

---

### 68. SCENARIO: Design a highly available, fault-tolerant 3-tier architecture on AWS.

**Answer:**
```
Region: us-east-1

VPC: 10.0.0.0/16

  ┌─── AZ: us-east-1a ─────────┬─── AZ: us-east-1b ─────────┐
  │                             │                             │
  │  Public Subnet              │  Public Subnet              │
  │  10.0.1.0/24               │  10.0.2.0/24               │
  │  [ALB Node]  [NAT-GW-1]    │  [ALB Node]  [NAT-GW-2]    │
  │                             │                             │
  │  Private App Subnet         │  Private App Subnet         │
  │  10.0.11.0/24              │  10.0.12.0/24              │
  │  [EC2/ECS] ← ASG           │  [EC2/ECS] ← ASG           │
  │                             │                             │
  │  Private DB Subnet          │  Private DB Subnet          │
  │  10.0.21.0/24              │  10.0.22.0/24              │
  │  [RDS Primary]             │  [RDS Standby]             │
  └─────────────────────────────┴─────────────────────────────┘

Components:
  - Route 53: DNS failover routing
  - CloudFront: CDN for static assets
  - WAF: Layer 7 protection on ALB
  - ALB: Distributes across app tier (multi-AZ)
  - ASG: Auto-scales app tier (min 2, max 20)
  - RDS Multi-AZ: Automatic failover for DB
  - ElastiCache: Redis for session/cache (multi-AZ)
  - S3: Static assets, backups
  - CloudWatch: Monitoring and alarms
  - AWS Backup: Automated backups
  - Security Groups: Layered security
  - Secrets Manager: Database credentials
```

---

### 69. SCENARIO: How do you reduce AWS costs for a dev environment that's only used during business hours?

**Answer:**
```bash
# Strategy 1: Schedule EC2 stop/start with Lambda + EventBridge
# 8 AM weekdays → Start instances
# 8 PM weekdays → Stop instances

# EventBridge rule
aws events put-rule \
  --name "stop-dev-instances" \
  --schedule-expression "cron(0 20 ? * MON-FRI *)" \
  --state ENABLED

# Lambda function to stop instances by tag
import boto3
def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    instances = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Environment', 'Values': ['dev']},
            {'Name': 'instance-state-name', 'Values': ['running']}
        ]
    )
    instance_ids = [i['InstanceId'] for r in instances['Reservations']
                    for i in r['Instances']]
    if instance_ids:
        ec2.stop_instances(InstanceIds=instance_ids)

# Strategy 2: Use Spot instances for dev (up to 90% savings)
# Dev can tolerate interruptions

# Strategy 3: RDS — stop when not in use
aws rds stop-db-instance --db-instance-identifier dev-database
# Note: RDS auto-starts after 7 days

# Strategy 4: ASG scheduled scaling
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name dev-asg \
  --scheduled-action-name scale-down-evening \
  --recurrence "0 20 * * 1-5" \
  --min-size 0 --max-size 0 --desired-capacity 0

# Strategy 5: Use ECS Fargate Spot for dev containers
# Strategy 6: Right-size instances (use t3.micro/small in dev)
# Strategy 7: Delete unattached EBS volumes and old snapshots
# Strategy 8: Set S3 lifecycle policies for dev buckets
```

---

### 70. SCENARIO: Your ALB is returning 502 Bad Gateway errors. How do you troubleshoot?

**Answer:**
```bash
# 502 = ALB received an invalid response from the backend target

# Step 1: Check ALB Access Logs
# Enable: ALB → Attributes → Access logs → S3 bucket
# Look for: 502 status, target_status_code, target response time

# Step 2: Check Target Group Health
aws elbv2 describe-target-health \
  --target-group-arn arn:aws:elasticloadbalancing:...
# Look for: unhealthy/draining targets

# Step 3: Check if application is running on the correct port
# ALB target group port must match app's listening port
curl http://instance-ip:8080/health    # Test directly

# Step 4: Check security group rules
# ALB security group must allow outbound to app port
# Instance security group must allow inbound from ALB SG

# Step 5: Check application logs for errors
aws logs tail /app/logs --follow

# Step 6: Check health check configuration
# Path, port, success codes must match app behavior

# Common causes:
# - App crashed on all instances
# - App listening on wrong port
# - App returns non-200 for health check path
# - Security group blocks ALB → Instance traffic
# - App timeout (ALB default: 60s)
# - App returns invalid HTTP response (missing headers)

# Fix timeout:
aws elbv2 modify-target-group-attributes \
  --target-group-arn arn:aws:... \
  --attributes Key=deregistration_delay.timeout_seconds,Value=30
```

---

### 71. SCENARIO: How do you set up a private microservices architecture where services communicate internally but only the API Gateway is public?

**Answer:**
```
Architecture:

Internet → Route 53 → CloudFront/WAF → API Gateway (public)
                                              ↓ (VPC Link)
VPC:
  Private Subnet:
    NLB (internal)
      ↓
    ECS/EKS Services (private, no public IP):
      - auth-service:8080
      - user-service:8081
      - order-service:8082
      - payment-service:8083
    
    Internal ALB for service-to-service:
      - services communicate via service discovery
      - AWS Cloud Map / Route 53 private hosted zone

  Data Subnet:
    RDS (private)
    ElastiCache (private)
```

```bash
# Service Discovery with Cloud Map
aws servicediscovery create-private-dns-namespace \
  --name internal.myapp.local \
  --vpc vpc-abc123

# Services register themselves
# auth-service → auth.internal.myapp.local:8080
# user-service → user.internal.myapp.local:8081

# API Gateway VPC Link to NLB
aws apigateway create-vpc-link \
  --name microservices-link \
  --target-arns arn:aws:elasticloadbalancing:...:loadbalancer/net/...

# Security:
# - All services in private subnets (no internet access)
# - Security groups: services only accept traffic from internal ALB
# - NACLs: restrict inter-subnet communication
# - IAM roles per service (least privilege)
# - Secrets Manager for credentials
# - VPC Endpoints for AWS service access (no internet needed)
```

---

*© AWS Services Interview Q&A — 71 Questions*
*Services: EC2 · S3 · VPC · RDS · ECR · EKS · ECS · EBS · EFS · IAM · Route 53 · ASG · ELB*
