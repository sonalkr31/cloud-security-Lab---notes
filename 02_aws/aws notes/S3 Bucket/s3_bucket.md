# Amazon S3 Notes & Cheat Sheet

## 1. Core Architecture: Buckets & Objects
Amazon S3 is a highly scalable object storage service (not a block or file system). It utilizes a flat, non-hierarchical architecture.

### Buckets (The Containers)
* **Global Namespace:** Bucket names must be globally unique across all AWS accounts worldwide (like a domain name).
* **Region-Specific:** You choose a specific AWS region when creating a bucket. Data never leaves that region unless you explicitly configure replication.
* **Bucket Limits:** By default, you can create up to 10,000 buckets per account.
* **Naming Rules:** 3–63 characters; lowercase letters, numbers, and hyphens only; no underscores; cannot end in a hyphen or look like an IP address.

### Objects (The Files)
* **Components:** An object consists of a Key (the file path string), Value (the raw bytes), Metadata (key-value pairs like Content-Type), and a Version ID.
* **Size Limits:** Objects can range from 0 bytes up to 5 Terabytes.
* **Multi-part Uploads:** Required for files larger than 5 GB, and highly recommended for any file over 100 MB to improve upload speed and reliability.
* **Virtual Folders:** S3 has no real folders. It simulates them using prefixes in the object key (e.g., `photos/2026/beach.jpg` is just a flat key containing slashes).

---

## 2. S3 Bucket Types
AWS categorizes S3 buckets based on performance and data types:

* **General Purpose Buckets:** The default type for virtually all standard use cases. Offers multi-AZ redundancy and scales infinitely.
* **Directory Buckets:** Built specifically for S3 Express One Zone. Designed for ultra-low, single-digit millisecond latency workloads.
* **Table & Vector Buckets:** Specialized bucket types optimized for tabular structured data (like database query formats) and machine learning vector embedding data.

---

## 3. Key Features to Know
* **Consistency Model:** S3 features strong read-after-write consistency for PUT and DELETE operations of all objects. Once a file is written or deleted, immediate subsequent reads will always reflect that change.
* **Versioning:** Protects against accidental deletion or overwrites by preserving history. Deleting an object creates a "Delete Marker." You can reverse this by deleting the marker.
* **Lifecycle Rules:** Automates cost savings by defining transitions (e.g., Move objects to Glacier after 30 days) or expiration policies (e.g., Delete logs after 90 days).
* **Static Website Hosting:** You can configure an S3 bucket to host a lightweight front-end website directly from a public URL.

---

## 4. Security & Access Control
S3 is private by default. Access can be managed via four mechanisms:

| Security Mechanism | Scope | Common Use Case |
|---|---|---|
| **IAM Policies** | User/Group level | Grants internal AWS users or applications permissions to access specific buckets. |
| **Bucket Policies** | Bucket level | Grants cross-account access, enforces SSL/HTTPS connectivity, or opens public access for static websites. |
| **Access Control Lists (ACLs)** | Object/Bucket level | Legacy method. Mostly disabled by default now in favor of Bucket Policies. |
| **Object Lock (WORM)** | Object level | Write Once, Read Many. Used for regulatory compliance. Comes in Governance mode (privileged users can override) and Compliance mode (even root cannot delete until the retention period expires). |

* **Encryption:** S3 supports Encryption in Transit (via TLS/HTTPS endpoints) and Server-Side Encryption at Rest via SSE-S3 (managed keys), SSE-KMS (custom key management), or SSE-C (customer-provided keys).

---

## 5. S3 Storage Classes Cheat Sheet
To save money, match your data's access patterns to the appropriate storage tier:

| Storage Class | Designed For | Min Storage Duration | Retrieval Fee | AZ Redundancy |
|---|---|---|---|---|
| **S3 Standard** | Frequent access, active data. | None | None | ≥ 3 AZs |
| **S3 Intelligent-Tiering** | Unknown or shifting access patterns. Automatically moves files to save money. | None | None | ≥ 3 AZs |
| **S3 Standard-IA** | Infrequent access but needs millisecond availability (e.g., backups). | 30 days | Per GB retrieved | ≥ 3 AZs |
| **S3 One Zone-IA** | Infrequent access, non-critical data that can be re-created if a data center destroys. | 30 days | Per GB retrieved | 1 AZ |
| **S3 Glacier Instant Retrieval** | Archive data accessed a few times a year but needed in milliseconds. | 90 days | Per GB retrieved | ≥ 3 AZs |
| **S3 Glacier Flexible Retrieval** | Archive data. Retrieval times range from 1–5 mins (Expedited) to 3–5 hours (Standard). | 90 days | Per GB retrieved | ≥ 3 AZs |
| **S3 Glacier Deep Archive** | Long-term retention (years). Lowest cost, retrieval takes 12–48 hours. | 180 days | Per GB retrieved | ≥ 3 AZs |
