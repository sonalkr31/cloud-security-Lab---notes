# Multi-Cloud Engineering & Security Architecture

A structured hands-on repository covering cloud infrastructure emulation, security patterns, automated deployments, and operational workflows across AWS, GCP, and Azure.

---

## Repository Structure

```text
cloud-security-notes/
├── 01_local_emulation/     # Zero-cost local cloud prototyping (Floci, LocalStack)
│   ├── docker-compose.yaml # Container definition
│   └── floci_guide.md      # Setup, S3 operations, DynamoDB workflows
├── 02_aws/                 # AWS IAM, VPC, and detection engineering
├── 03_gcp/                 # GCP IAM, Cloud Logging, and Workload Identity
├── 04_azure/               # Microsoft Entra ID and Sentinel configurations
└── README.md

```

---

## Active Modules

### 1. [Local Cloud Emulation with Floci](01_local_emulation/floci_guide.md)

* **Runtime**: Floci native binary (~24 ms boot, 13 MiB idle memory)
* **Target Services**: Amazon S3 (object lifecycle) & Amazon DynamoDB (NoSQL schemas)
* **Interface**: AWS CLI v2 via custom endpoint mapping (`http://localhost:4566`)

---

## Prerequisites

* **Runtime**: Docker Desktop (Apple Silicon ARM64 with Rosetta 2 support)
* **CLI**: `awscli` v2 via Homebrew
* **Tools**: Visual Studio Code, Git

```

---

