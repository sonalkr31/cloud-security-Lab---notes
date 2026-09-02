# Local AWS Cloud Emulation with Floci: A Complete Guide

Cloud computing is powerful, but developing directly on real AWS infrastructure can lead to unexpected billing shocks and requires constant internet connectivity. Local cloud emulation solves this by running a fully functional, offline replica of cloud services on your local machine.

This guide uses **Floci**, an MIT-licensed, open-source local cloud emulator that serves as a drop-in replacement for LocalStack. Built with Quarkus Native, Floci boots in approximately 24 milliseconds, consumes around 13 MiB of idle memory, and requires no authentication tokens or cloud accounts.

The following steps detail how to provision a complete local AWS environment—specifically covering S3 storage and DynamoDB NoSQL databases—optimized for an Apple Silicon architecture (M4).

---

## Phase 1: Virtualization Engine Setup

Floci runs inside Docker, a containerization platform that packages software in isolated environments.

### 1. Install Docker Desktop

For macOS users, Homebrew is the cleanest package manager to install the native ARM64 Docker binary:

```bash
brew install --cask docker

```

After installation, launch Docker from the Applications folder and accept the system configuration prompts to initialize the daemon.

### 2. Enable Rosetta 2 (Apple Silicon Specific)

While Floci runs natively on ARM64 processors, some older or specialized container images you may pull in the future only exist for Intel (x86_64) architectures. Rosetta 2 translates these instructions seamlessly on Apple Silicon.

```bash
softwareupdate --install-rosetta --agree-to-license

```

---

## Phase 2: Provisioning the Local Cloud

Instead of running a long terminal command every time you want to start the emulator, best practice dictates using a `docker-compose.yaml` blueprint. This keeps infrastructure as code.

### 1. Create a Dedicated Workspace

```bash
mkdir ~/my_local_cloud
cd ~/my_local_cloud

```

### 2. Define the Emulator Blueprint

Create the configuration file:

```bash
nano docker-compose.yaml

```

Paste the following YAML configuration. This pulls the latest Floci image and maps port `4566`, the standard local AWS emulation port:

```yaml
services:
  floci:
    image: floci/floci:latest
    ports:
      - "4566:4566"

```

*(Save and exit using `Ctrl + O`, `Enter`, `Ctrl + X`)*

### 3. Boot the Control Plane

Start the environment in detached mode (background):

```bash
docker compose up -d

```

Verify the container is healthy:

```bash
docker ps

```

---

## Phase 3: The Command Interface

The AWS Command Line Interface (CLI) is the primary tool for interacting with cloud resources.

### 1. Install the CLI

```bash
brew install awscli

```

### 2. Configure Dummy Credentials

The AWS CLI mandates security credentials. Because Floci runs locally and does not validate authentication, you must provide dummy variables to satisfy the CLI's internal checks:  // run the command one by one .

```bash
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1

```

### 3. Route Traffic Locally

By default, the CLI attempts to reach Amazon's production servers. Force it to target your local emulator for the current session: . Use this command to use your locally url .

```bash
export AWS_ENDPOINT_URL=http://localhost:4566

```

---

## Phase 4: Object Storage (Amazon S3)

Amazon Simple Storage Service (S3) is the backbone of cloud object storage.

### 1. Create a Bucket

```bash
aws s3 mb s3://my-practice-bucket

```

### 2. Upload Data

Cloud objects are immutable. Create a local text file without using quotes to avoid terminal parsing errors, then push it to S3:

```bash
echo Hello from Floci > hello.txt
aws s3 cp hello.txt s3://my-practice-bucket/

```

### 3. Verify and Update Data

To update an object, upload a file with the exact same name to overwrite the existing asset:

```bash
echo This is the new updated text > hello.txt
aws s3 cp hello.txt s3://my-practice-bucket/
aws s3 ls s3://my-practice-bucket/

```

### 4. Infrastructure Teardown

S3 requires buckets to be empty before deletion to prevent accidental data loss. command -rm to remove file, rb to remove file .

```bash
aws s3 rm s3://my-practice-bucket/hello.txt
aws s3 rb s3://my-practice-bucket/

```

---

## Phase 5: NoSQL Databases (DynamoDB)

DynamoDB is a high-performance NoSQL database frequently utilized in cybersecurity event logging and web applications.

### 1. Define the Table Schema

Unlike S3, a database requires a predefined structure—specifically the Primary Key. The following command creates a `Users` table partitioned by a `Username` string (`S`):

```bash
aws dynamodb create-table \
  --table-name Users \
  --attribute-definitions AttributeName=Username,AttributeType=S \
  --key-schema AttributeName=Username,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

```

### 2. Insert Records

Insert a JSON payload defining the user and their associated course:

```bash
aws dynamodb put-item \
  --table-name Users \
  --item '{"Username": {"S": "Kumar_Sonal"}, "Course": {"S": "Cybersecurity"}}'

```

### 3. Query the Database

Execute a full table scan to verify data persistence:

```bash
aws dynamodb scan --table-name Users

```

---

## Troubleshooting Guide

**The `dquote>` Trap:**
If your terminal shows `dquote>`, it is waiting for a closing quotation mark. This frequently happens when copying and pasting "smart quotes" from websites. Press `Ctrl + C` to break out, and rewrite the command using standard straight quotes or omit quotes entirely where possible.

**Command Not Found (`aws`):**
Errors like `zsh: command not found: ws` indicate a typo in the executable name. Ensure the command begins strictly with `aws`.

**ParamValidation Errors:**
If the CLI rejects a command (e.g., `invalid choice 'cp'`), ensure the specific AWS service name (`s3`, `dynamodb`) precedes the action command. The structure is always `aws [service] [action]`.