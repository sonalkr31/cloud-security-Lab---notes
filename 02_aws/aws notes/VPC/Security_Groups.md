#  Security Groups and Networks



A Security Group acts as a virtual firewall that controls inbound and outbound network traffic for individual AWS resources (such as an EC2 instance).Unlike a Network ACL (NACL), which protects the entire subnet boundary, a Security Group operates directly at the instance level.

## Key Functions and CharacteristicsStateful Filtering: 
*They automatically remember connection states. If you create a rule to allow inbound traffic on a specific port, the return outbound traffic is automatically allowed, regardless of your outbound rules.

## Allow Rules Only: 
* You can only specify rules that permit traffic. By default, all inbound traffic is blocked until you add a rule to allow it, and you cannot create explicit "deny" rules.
## Evaluates All Rules: 
* Unlike NACLs, which evaluate rules in numerical order, a Security Group evaluates all available rules simultaneously to decide whether to permit traffic.



# AWS Network ACL (NACL) vs. Security Group (SG)

| Feature | Network ACL (NACL) | Security Group (SG) |
| --- | --- | --- |
| **Primary Level** | **Subnet level** (Acts as a perimeter fence for the whole subnet). | **Instance / ENI level** (Acts as a personal guard for an individual resource). |
| **State Nature** | **Stateless:** Return traffic must be explicitly allowed by an outbound rule. | **Stateful:** Return traffic is automatically allowed, regardless of outbound rules. |
| **Rule Capabilities** | Supports both **Allow and Deny rules** (great for explicitly blocking specific malicious IPs). | Supports **Allow rules only** (everything else is implicitly denied). |
| **Rule Processing** | Evaluated **sequentially** in numerical order (lowest rule number processed first). | All rules are evaluated **simultaneously** before traffic is permitted. |
| **Default Settings** | **Default NACL** allows all traffic. **Custom NACLs** deny all traffic until configured. | **Default SG** allows all outbound traffic but blocks all external inbound traffic. |
| **Resource Attachment** | Automatically applies to all instances deployed within the associated subnet. | Must be explicitly assigned to individual instances or network interfaces. |
| **Target Referencing** | Rules can only reference IP addresses or CIDR blocks. | Rules can reference IP addresses, CIDR blocks, or other Security Groups. |
| **Number of Rules** | Limited quota (typically 20 inbound and 20 outbound rules, adjustable up to 40). | Higher quota (typically 60 inbound and 60 outbound rules per group). |
| **Order of Defense** | **First line of defense** for inbound traffic entering the subnet. | **Second line of defense** for inbound traffic reaching the actual instance. |




```text
###  Practical Scenario: Protecting a Web Server
1. The NACL (Subnet Firewall):** Evaluates the incoming traffic first. It allows HTTP/HTTPS traffic from `0.0.0.0/0` (everyone) but has a specific Deny rule for `203.0.113.50` (a known malicious IP).
2. The Security Group (Instance Firewall):** After the NACL allows the traffic, the Security Group evaluates it at the EC2 level. It only has Allow rules for port 80 (HTTP) and 443 (HTTPS). 
3. The Return Trip:** Because the SG is stateful, the EC2 instance's response automatically flows back to the user. However, because the NACL is stateless, you must have an outbound NACL rule explicitly allowing traffic on ephemeral ports (1024-65535) so the response can successfully leave the subnet.

```


[ The Public Internet ]
                 │
                 ▼ (User requests website)
══════════════════════════════════════════════════════════════════
  AWS VPC BOUNDARY
                 │
         [ Internet Gateway ]
                 │
──────────────────────────────────────────────────────────────────
  PUBLIC SUBNET BOUNDARY (e.g., 10.0.1.0/24)

    🛡️ 1. NETWORK ACL (The Subnet Firewall - Stateless)
          ├─ INBOUND RULES (Evaluated numerically): 
          │    ❌ Rule 90:  Deny ALL from 203.0.113.50 (Malicious IP)
          │    ✅ Rule 100: Allow HTTP (Port 80) from 0.0.0.0/0
          │    ✅ Rule 110: Allow HTTPS (Port 443) from 0.0.0.0/0
          │
          ├─ OUTBOUND RULES (Required for Return Traffic!):
          │    ✅ Rule 100: Allow Ephemeral Ports (1024-65535) 
          │                 out to 0.0.0.0/0
          │
                 ▼ (Clean traffic passes through)
                 │
    ┌────────────────────────────────────────────────────────┐
    │  EC2 INSTANCE (The Web Server)                         │
    │                                                        │
    │ 🛡️ 2. SECURITY GROUP (The Instance Firewall - Stateful)│
    │        ├─ INBOUND RULES:                               │
    │        │    ✅ Allow HTTP (Port 80) from 0.0.0.0/0     │
    │        │    ✅ Allow HTTPS (Port 443) from 0.0.0.0/0   │
    │        │    (The malicious IP never reaches here)      │
    │        │                                               │
    │        └─ OUTBOUND RETURN TRAFFIC:                     │
    │             (No rules needed for the user's response)  │
    │             (Stateful: Response is auto-allowed!)      │
    └────────────────────────────────────────────────────────┘