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
