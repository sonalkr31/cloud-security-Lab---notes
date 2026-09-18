# Ultimate AWS CLI Enterprise Cheat Sheet

A curated repository of production-grade **AWS CLI** one-liners, automation scripts, and diagnostic queries. Structured for quick access in VS Code, system administration, and deployment via GitHub.

---

## 1. Global Setup, Context Configuration & Profiles

Before running infrastructure queries, configure the CLI to switch context efficiently between authentication schemes, roles, and output formats.

### Authentication & Profile Initializations
```bash
# Interactive setup for standard IAM User access keys
aws configure

# Named profile creation to separate development, staging, and production environments
aws configure --profile production

# Force an explicit session authentication refresh when using AWS SSO / IAM Identity Center
aws sso login --profile enterprise-dev

# Quick validation of your active identity context (Answers: Who am I logged in as?)
aws sts get-caller-identity

# Dynamic environment variable profile override (Bypasses active default configurations)
export AWS_PROFILE="production"
export AWS_DEFAULT_REGION="us-east-1"
```

### Advanced Formatting & Parsing with `--query` and `jq`
AWS CLI uses **JMESPath** for server-side filtering via `--query`. Combine this with client-side `jq` processing to extract specific data arrays.
```bash
# Output format options: json, text, table
aws ec2 describe-instances --output table

# Server-side projection: Extract only instance IDs and states across all regions
aws ec2 describe-instances --query "Reservations[*].Instances[*].{ID:InstanceId,State:State.Name}" --output json

# Client-side filtering: Pipe raw JSON to jq to find active resources running on specific AMIs
aws ec2 describe-instances | jq '.Reservations[].Instances[] | select(.State.Name=="running") | .ImageId'
```

---

## 2. Amazon S3 (Simple Storage Service) Data & Management Operations

High-performance commands for object store interactions, bulk data parsing, and access auditing.

### Bucket Creation & Structure Control
```bash
# Create a standard general-purpose bucket in a non-default region (requires location constraint)
aws s3api create-bucket \
    --bucket enterprise-vault-storage-2026 \
    --region eu-west-1 \
    --create-bucket-configuration LocationConstraint=eu-west-1

# Apply a comprehensive Public Access Block to secure data (Secures bucket from accidental leaks)
aws s3api put-public-access-block \
    --bucket enterprise-vault-storage-2026 \
    --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Enable bucket versioning to protect against accidental object deletion or overwrites
aws s3api put-bucket-versioning \
    --bucket enterprise-vault-storage-2026 \
    --versioning-configuration Status=Enabled
```

### High-Speed Data Engineering & Sync
```bash
# High-speed parallel synchronization of local directory to S3 (Uploads modified/new assets only)
aws s3 sync ./dist s3://enterprise-vault-storage-2026/site-deploy/ --delete

# Large file upload optimization: Tune configuration variables for Multi-part Upload speeds
aws configure set default.s3.max_concurrent_requests 20
aws configure set default.s3.multipart_threshold 64MB

# Secure down-stream transfer: Generate a time-limited pre-signed URL valid for 30 minutes (1800 seconds)
aws s3 presign s3://enterprise-vault-storage-2026/reports/audit-q3.pdf --expires-in 1800
```

### Forensic Inventory Queries
```bash
# Calculate total aggregate storage size and object volume inside an S3 prefix
aws s3 ls s3://enterprise-vault-storage-2026/logs/ --recursive --human-readable --summarize

# Identify and list uncompleted Multi-part Uploads (Hidden sources of rising storage costs)
aws s3api list-multipart-uploads --bucket enterprise-vault-storage-2026

# Forcefully empty and remove a bucket containing thousands of objects and versions
aws s3 rb s3://enterprise-vault-storage-2026 --force
```

---

## 3. Network Architecture, Security Groups & VPC Diagnostics

Real-time infrastructure troubleshooting tools to resolve connectivity problems across Layer 3 and Layer 4 boundaries.

### Network Interface & IP Auditing
```bash
# Find the exact Network Interface (ENI) matching a private IP address within a VPC
aws ec2 describe-network-interfaces \
    --filters "Name=addresses.private-ip-address,Values=10.0.4.112" \
    --query "NetworkInterfaces[*].{ENI:NetworkInterfaceId,Description:Description,AttachedTo:Attachment.InstanceId}"

# Extract all public IPs assigned to active load balancers or EC2 instances
aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].{ID:InstanceId,PublicIP:PublicIpAddress,Subnet:SubnetId}"
```

### Security Group Rule Auditing
```bash
# Audit Warning: Find all Security Groups containing globally exposed SSH or RDP access rules
aws ec2 describe-security-groups \
    --filters "Name=ip-permission.cidr,Values=0.0.0.0/0" "Name=ip-permission.to-port,Values=22,3389" \
    --query "SecurityGroups[*].{GroupID:GroupId,Name:GroupName}"

# Instantly strip a broad rule from an existing group to remediate a security alert
aws ec2 revoke-security-group-ingress \
    --group-id sg-0123456789abcdef0 \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

---

## 4. Amazon EC2 & Systems Manager (SSM) Host Automations

Fleet management control plane tools to handle scaling, tracking, and secure connections.

### Advanced Provisioning & Lifecycle Control
```bash
# Provision a Linux instance, inject User Data script execution, and apply explicit resource tags
aws ec2 run-instances \
    --image-id ami-0c7217cdde317cfec \
    --count 1 \
    --instance-type t3.medium \
    --key-name prod-ssh-key \
    --security-group-ids sg-0a1b2c3d4e5f6g7h8 \
    --subnet-id subnet-0123456789abcdef0 \
    --user-data file://bootstrap_script.sh \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Environment,Value=Production},{Key=Role,Value=AppServer}]'

# Programmatic termination targeting a malfunctioning microservice node
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0
```

### Systems Manager (SSM) Secure Session Access
```bash
# Establish a secure interactive terminal console connection (Replaces legacy exposed SSH bastions)
aws ssm start-session --target i-0123456789abcdef0

# Run a shell command across an entire fleet simultaneously using tag targeting
aws ssm send-command \
    --targets "Key=tag:Role,Values=AppServer" \
    --document-name "AWS-RunShellScript" \
    --parameters 'commands=["systemctl restart nginx"]'
```

---

## 5. IAM (Identity & Access Management) Security Hardening

Commands to audit credentials, access privileges, and key rotational states.

### Identity & Privilege Assessment
```bash
# Generate a complete account credential report (Triggers background AWS audit generation)
aws iam generate-credential-report
aws iam get-credential-report --output text --query "Content" | base64 --decode > report.csv

# List all access keys assigned to an employee account along with creation date strings
aws iam list-access-keys --user-name j.doe@company.com

# Deactivate a vulnerable or compromised Access Key immediately without deleting history
aws iam update-access-key \
    --user-name j.doe@company.com \
    --access-key-id AKIAIOSFODNN7EXAMPLE \
    --status Inactive
```

---

## 6. CloudWatch Logs, Insights & Troubleshooting Diagnostics

Commands to query and extract active infrastructure logs directly into your terminal.

### Real-Time Ingestion Tracking
```bash
# Stream and follow live application stdout streams directly within your shell console
aws logs tail /aws/lambda/production-order-processor --follow

# Search across error log groupings over a targeted time boundary (1h = last hour)
aws logs filter-log-events \
    --log-group-name "/aws/ecs/production-cluster-api" \
    --filter-pattern "HTTP 500" \
    --start-time $(date -d '1 hour ago' +%s%3N)
```

---

## 7. AWS CloudTrail, Config & Logging Guardrails

Commands to initialize and verify logging workflows.

### Operational Logging Deployment
```bash
# Initialize a Multi-Region CloudTrail audit log to record your infrastructure actions
aws cloudtrail create-trail \
    --name global-enterprise-audit-trail \
    --s3-bucket-name enterprise-vault-storage-2026 \
    --is-multi-region-trail \
    --enable-log-file-validation

# Start processing active configuration changes across your region using AWS Config
aws configservice start-configuration-recorder --configuration-recorder-name default
```

---

## 8. Ultimate Troubleshooting Syntax Patterns

Common solutions for errors encountered when operating the AWS CLI tool chain.

### Error Scenario 1: Access Denied / Explicit Denies
* **Problem Statement:** Command throws `An error occurred (AccessDenied) when calling the ListObjectsV2 operation: Access Denied`.
* **Troubleshooting Action Pipeline:**
  1. Verify active token validation scopes via `aws sts get-caller-identity`.
  2. Append `--debug` to the command string. This lists the signed header context and shows whether the block is originating from an **IAM Policy**, a **Bucket Policy**, or an **Organizations Service Control Policy (SCP)**.

### Error Scenario 2: Region Incongruence
