# AWS Auditing & Logging: CloudTrail, VPC Flow Logs, & AWS Config

Comprehensive, deep-dive reference notes for tracking API activity, monitoring network traffic, and enforcing resource compliance across AWS infrastructure.

---

## 1. AWS CloudTrail (API Activity & Governance)
AWS CloudTrail is a service that enables governance, compliance, operational auditing, and risk auditing of your AWS account. It answers the questions: *Who requested what resource, from where, and when?*

### Core Architecture & Mechanics
* **The Audit Trail:** Every action taken via the AWS Management Console, AWS SDKs, command line tools, or direct infrastructure APIs is recorded as an event.
* **Global vs. Regional Trails:** 
  * **Regional Trail:** Delivers events only from the specific region it is created in.
  * **Multi-Region Trail:** Configured once to automatically replicate settings across all current and future regions in an account. This is a critical security best practice to catch unauthorized resources launched in dormant regions.
* **Log Storage & Integrity:**
  * **S3 Delivery:** Logs are aggregated and pushed to a designated S3 bucket as compressed JSON files.
  * **Log File Integrity Validation:** Uses cryptographic hashing (SHA-256) and digital signing (RSA) to create a digest file. This allows you to mathematically prove if a log file was modified, deleted, or tampered with after delivery.

### CloudTrail Event Types
* **Management Events (Control Plane):**
  * Tracks structural operations that modify or view your infrastructure setup.
  * *Examples:* `CreateBucket`, `RunInstances`, `CreateUser`, `AttachPolicy`.
  * *Pricing:* Enabled **by default** across all accounts; the first copy of management events in each region is free. Holds a rolling **90-day history** in the Event History console.
* **Data Events (Data Plane):**
  * Tracks high-volume data-handling activities inside resources.
  * *Examples:* S3 object-level actions (`GetObject`, `PutObject`), AWS Lambda function executions (`InvokeFunction`), Amazon DynamoDB item-level mutations.
  * *Pricing:* Disabled **by default** due to massive throughput scale. These incur additional data ingestion fees.
* **CloudTrail Insights:**
  * Uses machine learning to analyze management event patterns and establish an operational baseline.
  * Automatically flags anomalies such as a sudden spike in `TerminateInstances` APIs or unusual IAM policy modifications.

---

## 2. VPC Flow Logs (Network Traffic Tracking)
VPC Flow Logs capture network traffic metadata going to and from network interfaces (ENIs) within your Virtual Private Cloud. 

### Core Mechanics & Limits
* **Out-of-Band Capture:** Log collection happens outside the actual path of your network data. It does not affect network throughput, latency, or bandwidth.
* **Collection Levels:** Can be enabled and scoped at three granular hierarchies:
  1. **VPC Level:** Collects telemetry for all network interfaces within the entire VPC boundary.
  2. **Subnet Level:** Collects telemetry for all network interfaces inside that specific subnet.
  3. **Network Interface (ENI) Level:** Targets a single specific interface (e.g., an EC2 instance's primary ENI, or an RDS endpoint).
* **Traffic States:** Categorizes traffic statuses into:
  * **ACCEPT:** Traffic permitted by both security groups and network ACLs (NACLs).
  * **REJECT:** Traffic blocked by either a security group (stateful) or an ACL rule (stateless).

### Architecture Destinations
* **Amazon CloudWatch Logs:** Used for real-time analysis, metric filters, and triggering alarms (e.g., alert if REJECT packet count spikes).
* **Amazon S3:** Used for cost-effective, long-term raw retention. Perfect for running SQL analytics via **Amazon Athena**.
* **Amazon Kinesis Data Firehose:** Streams flow log data directly into third-party SIEM providers (like Splunk, Datadog, or an ELK stack).

### Key Use Cases
* **S3 Gateway Endpoint Auditing:** When traffic routes privately to Amazon S3 via a Gateway Endpoint, VPC flow logs allow you to verify traffic remains entirely off the public internet by checking destination IP patterns.
* **Troubleshooting Connection Issues:** Identifying if security groups are over-restrictive by filtering for high concentrations of `REJECT` packets on known operational ports.

---

## 3. AWS Config (Configuration Management & Compliance)
AWS Config continuously monitors, records, and evaluates the configuration states of your AWS resources. It serves as an automated auditor tracking changes over time.

### Core Architecture Components
* **Configuration Recorder:** The background engine that detects structural adjustments to resources (e.g., an EBS volume size change, an S3 bucket policy adjustment). It records these changes as a **Configuration Item (CI)**.
* **Configuration Timeline:** A visual "time machine" detailing every change made to a specific resource, what user triggered it (via a CloudTrail tie-in), and how the asset's relationship with other resources changed.
* **Configuration Stream:** A live feed of configuration changes published straight to an **Amazon SNS** topic for immediate external alerting.

### Rule Evaluations & Triggers
AWS Config matches recorded resource configurations against compliance policies called **Rules**.

* **AWS Managed Rules:** Pre-built rules managed by AWS based on organizational cloud design best practices.
  * *Examples:* `s3-bucket-public-read-prohibited`, `iam-password-policy`, `encrypted-volumes`.
* **Custom Rules:** Custom business logic authored using **AWS Lambda** functions to audit non-standard infrastructure designs.
* **Evaluation Triggers:**
  * **Configuration Changes:** Evaluated instantaneously whenever a targeted resource is created, altered, or destroyed.
  * **Periodic / Schedule-Based:** Evaluated at a defined, repeating interval (e.g., every 1, 3, 6, 12, or 24 hours) to verify static states.

### Remediation Framework
When a resource drops out of compliance (e.g., a bucket becomes public), AWS Config can automatically resolve the issue:
* **AWS Systems Manager (SSM) Automation:** Config links directly with SSM Automation playbooks.
* **Auto-Remediation:** If the rule `s3-bucket-public-read-prohibited` fails, it can fire an SSM document that strips the public read permissions automatically within seconds of detection, eliminating reliance on manual operator intervention.

---

## 4. Deep-Dive Direct Comparisons
Use this quick overview to distinguish between these overlapping observability systems:

| Criteria | AWS CloudTrail | VPC Flow Logs | AWS Config |
|---|---|---|---|
| **Primary Scope** | User/IAM/API Activities | Network Traffic Layer (Layer 3/4) | Resource Configuration & Compliance |
| **Monitors What** | **Who** executed **what** API request | **Source/Destination** IPs, ports, packets | **Configuration state** changes over time |
| **Granularity** | Account, Region, or Global | VPC, Subnet, or ENI level | Individual AWS Resource level (CIs) |
| **Key Use Case** | Security forensic audits & attribution | Network troubleshooting & firewalls | Compliance reporting & configuration tracking |
| **Detection Mode** | Point-in-time discrete events | Continuous raw flow stream data | State history timeline capture |

---

## 5. Implementation Cheat Sheet (AWS CLI)
Quick execution workflows for establishing core governance configurations.

### AWS CloudTrail Setup
Create a globally scalable multi-region audit trail with built-in log integrity controls:

```bash
# 1. Create the S3 bucket designated for audit log storage
aws s3api create-bucket \
    --bucket my-company-audit-logs-prod \
    --region us-east-1

# 2. Create a secure, multi-region CloudTrail instance
aws cloudtrail create-trail \
    --name production-global-audit-trail \
    --s3-bucket-name my-company-audit-logs-prod \
    --is-multi-region-trail \
    --enable-log-file-validation

# 3. Initialize the trail to begin capturing active management planes
aws cloudtrail start-logging \
    --name production-global-audit-trail
```

### VPC Flow Logs Setup
Deploy network capture across a target VPC directly to an S3 log collection sink:

```bash
# Create a flow log capturing ALL traffic across a designated VPC
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-0a1b2c3d4e5f6g7h8 \
    --traffic-type ALL \
    --log-destination-type s3 \
    --log-destination arn:aws:s3:::my-company-audit-logs-prod/vpc-flows/
```

---

## 6. Parsing Network Logs via Amazon Athena
When VPC Flow Logs are exported to Amazon S3, you can query massive volumes of raw logs instantaneously using SQL patterns.

### Step 1: DDL Table Creation
Execute this schema statement inside your Amazon Athena editor window (adjust the `LOCATION` parameter to match your S3 logging sink structure):

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS vpc_flow_logs (
  version int,
  account_id string,
  interface_id string,
  srcaddr string,
  dstaddr string,
  srcport int,
  dstport int,
  protocol bigint,
  packets bigint,
  bytes bigint,
  start_time bigint,
  end_time bigint,
  action string,
  log_status string
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ' '
LOCATION 's3://my-company-audit-logs-prod/vpc-flows/AWSLogs/123456789012/vpcflowlogs/us-east-1/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

### Step 2: Diagnostic Audit Queries
Run these analytics queries to identify anomalous patterns or investigate routing behavior.

#### Query top 10 rejected connections on known sensitive ports
```sql
SELECT srcaddr, dstport, count(*) as reject_count
FROM vpc_flow_logs
WHERE action = 'REJECT'
  AND dstport IN (22, 3389, 443, 80)
GROUP BY srcaddr, dstport
ORDER BY reject_count DESC
LIMIT 10;
```




==========================================


## 8. Advanced Monitoring, Roles & Managed Querying

### Amazon CloudWatch Metric Filters for CloudTrail
Metric filters scan incoming CloudTrail log events (sent to CloudWatch Logs) for specific terms, phrases, or values. When a pattern match occurs, it increments a CloudWatch metric, which can trigger immediate CloudWatch Alarms to alert your team via SNS.

#### Scenario 1: Authorization Failures (Unauthorized Actions)
Detects brute-force attempts, compromised credentials, or misconfigured applications attempting unauthorized actions.
* **Filter Pattern:** `{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied") }`
* **Metric Name:** `AuthorizationFailures`
* **Metric Namespace:** `CloudTrailMetrics`

#### Scenario 2: Root Account Usage
Alerts security teams immediately whenever the root user performs an action (a high-severity compliance violation in standard operations).
* **Filter Pattern:** `{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }`
* **Metric Name:** `RootAccountUsage`
* **Metric Namespace:** `CloudTrailMetrics`

#### Scenario 3: CloudTrail Deletion or Interruption
Detects an attacker trying to cover their tracks by altering or disabling CloudTrail logging.
* **Filter Pattern:** `{ ($.eventName = StopLogging) || ($.eventName = UpdateTrail) || ($.eventName = DeleteTrail) }`
* **Metric Name:** `CloudTrailAlterations`
* **Metric Namespace:** `CloudTrailMetrics`

---

### IAM Policy Permissions for AWS Config Remediation
When AWS Config triggers auto-remediation via an AWS Systems Manager (SSM) Automation document, it must assume an IAM role (`ConfigRemediationSSMRole`). This role must contain a trust relationship with Systems Manager and explicit, least-privilege permissions to modify the non-compliant resource.

#### Trust Policy (Assume Role)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "://amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### IAM Permissions Policy (`AWS-CloseS3Bucket` Example)
This policy allows the remediation role to verify bucket configurations and apply a strict `PutBucketPublicAccessBlock` API action to isolate the public bucket.
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "s3:PutBucketPublicAccessBlock",
        "s3:GetBucketPublicAccessBlock",
        "s3:GetBucketPolicyStatus",
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::*"
    },
    {
      "Sid": "SSMExecutionPermissions",
      "Effect": "Allow",
      "Action": [
        "ssm:StartAutomationExecution",
        "ssm:GetAutomationExecution"
      ],
      "Resource": "*"
    }
  ]
}
```

---

### AWS CloudTrail Lake vs. Athena Log Parsing
Instead of managing the infrastructure pipeline required to query raw S3 logs via Amazon Athena (which requires manual DDL schema definitions, partition tuning, and S3 bucket architecture setups), you can leverage **AWS CloudTrail Lake**.

#### Core Trade-Off Matrix

| Feature | Amazon Athena (with S3) | AWS CloudTrail Lake |
| :--- | :--- | :--- |
| **Setup Complexity** | High (Requires S3 buckets, manual Glue/Athena DDL configuration, and partition logic) | Zero (Turn-key managed event data store natively optimized for queries) |
| **Performance Integration**| Fast for generalized querying; reliant on optimal S3 partitioning | Fast, out-of-the-box analytical engine pre-tuned specifically for CloudTrail schemas |
| **Query Engine** | Standard Presto/Trino SQL engine | Built-in ANSI SQL parsing query interface |
| **Retention Strategy** | Managed manually by S3 Lifecycle rules (move to Glacier Deep Archive) | Built-in immutable retention retention parameters up to 7 years |
| **Cost Driver** | Charged per GB of data scanned during queries + standard S3 storage fees | Charged per GB of data ingested and stored + per GB scanned |

#### CloudTrail Lake SQL Query Syntax Examples
CloudTrail Lake uses a native analytical table structure called an **Event Data Store ID** (`eds-id`).

##### Identify the exact IAM user who modified an S3 bucket policy:
```sql
SELECT
    eventTime,
    userIdentity.arn AS user_arn,
    awsRegion,
    requestParameters
FROM
    example-eds-id-12345
WHERE
    eventName = 'PutBucketPolicy'
    AND eventTime > '2026-09-01 00:00:00'
```

##### List the top 5 most frequently executed API operations over the last week:
```sql
SELECT
    eventName,
    count(*) as total_invocations
FROM
    example-eds-id-12345
WHERE
    eventTime > date_add('day', -7, current_timestamp)
GROUP BY
    eventName
ORDER BY
    total_invocations DESC
LIMIT 5;
```
