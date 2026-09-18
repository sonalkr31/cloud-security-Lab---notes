#          ELASTIC IP  (EIP)

An AWS Elastic IP is a static public IPv4 address designed for dynamic cloud computing that stays assigned to your account until you release it.

* AWS sets a default limit of five (5) Elastic IP addresses per AWS account, per region
You can increase your AWS Elastic IP (EIP) limit by requesting a quota increase through the Service Quotas console. 
By default, AWS limits your account to five (5) Elastic IP addresses per region. Quotas are region-specific, meaning an increase in one region does not apply to others

## Key FeaturesStatic Address: 
Does not change when you stop and restart your instance.

Quick Remapping:
* Masks instance or software failures by redirecting   traffic to a different instance instantly.

 *   Decoupled Architecture: The public IP address remains entirely stable on the client side.
    
*   Instant Shift: If your primary application instance or its software crashes, you can programmatically or manually reassign that identical public IP to a standby or backup instance.
    
*    Zero DNS Wait: Because the IP itself never changes, you bypass the typical internet propagation delays associated with updating standard Domain Name System (DNS) records, masking the infrastructure failure from your end users almost instantly

## NetworkInterface Property:
*  Attached to an Elastic Network Interface (ENI) rather than directly to a single instance.
  
 * Pricing RulesFree: One Elastic IP is free while attached to a running EC2 instance.
  
 * Paid: Incurs a small hourly charge if it is unassociated, attached to a stopped instance, or if you hold multiple unused addresses


  ## All Rules for easatic ip :- 

* Limits & Assignment: Max 5 Elastic IPs per region; can only attach to one instance/network interface at a time; can freely move between them.
* IPv6 Support: Not supported (IPv4 only).
* Scope: Elastic IPs are regional; use Global Accelerator for global static IPs.
* IP Sourcing: Can be Amazon-provided (restricted by network border groups) or BYOIP (Bring Your Own IP).
* Internet Access: Requires open Security Groups / Network ACLs and an Internet Gateway for two-way traffic.
* Public IP Behavior: Associating an Elastic IP releases the original public IP to the AWS pool. Disassociating it automatically assigns a new public IP within minutes (unless a second network interface is attached).
* Tagging: Supports resource tagging, but tags are lost if you recover a released Elastic IP.
* Cost Tracking: Public IPv4 addresses support cost allocation tags (takes up to 24 hours to appear for activation in AWS Billing).
* Cost Explorer Filters: Track active Elastic IP costs using PublicIPv4InUseAddress (Hrs) and idle/unassociated costs using PublicIPv4IdleAddress (Hrs).





 * How to Create and AssignOpen the Amazon EC2 Console.