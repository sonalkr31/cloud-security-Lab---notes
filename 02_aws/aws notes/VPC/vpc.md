#            VPC

## Wht is VPC?
      A private isolated network within the AWS cloud where we can launch and manage our resources  securely.
                VPC --- Virtual private Cloud .

## Why we need  VPC?
       To securely isolate and control network environment. we get additional layer of sefety using VPC . 

    ## What happens when we create VPC?
        When we create VPC , we specify a CIDR block that defines the IP address range for the entire VPC,
            for ex- 10.0.0.0/16 - this blocks allows for 65536  Ip addresses (built in reality , 65531 usable addresses )
                    CIDR (class inter domain routing ) is a method for allocating IP addresses and routing Internet protocol (IP ) packets

   ## CIDR Block allocatiom 
         we can specify a range of IP (10.0.0.0/16 - 65536 ) addresses (CIDR block) within the VPC's IP addresses range for the subnet
                 THIS determines the pool if IP addresses available for instances in the subnet. 

                 the range should be under the VPC.
                        
                        
                        ## To understand this better go to website cidr.xyz  , i is an interacting website.
                                there is default VPC whic provided and manged by aws through which  we perform or all task.




## What is Subnet?
    A subnet is a smaller , segmented part of a larger network that isolates and organize devices within a speecific Ip adress range.   
                    VPC 100 IPs
                        |
                 |               | 
    public-subnets 50 IPs         private-subnets. 50 IPs
    ![alt text](image.png)


    ## Note:- WE CANNOT USE VPC DIRECT . so we have to use subnet so CIDR 
    Inside the network we create RESOURCE (ex-EC2 instance )

        EC2 instance is created under the subnet
        as shown in the figure

        Look inn the photo as we public subnet .
            the range we are providing of the subnet will be the same ip asssigned . you can try in aws vpc

            NOTE:- WE CAN ASSIGN MULTIPLE SUBNET , EVEN WE CAN CHANGE THE THE NAME OF OUR SUBNETS

            Why more subnet --- because we in our website which is accessible to public directly and may our main data in private . that's why

              FOR EX--- we put our frontend in public and data (database)in private so user or client doesnt have the access of data directly.
    
          NOTE 

  ####  WE CAN CREATE MULTIPLE INSTANCE IN ONE SUBNET , as shown in the figure below. 
THERE ARE MULTIPLE CASES ,--- WE CAN CREATE MULTIPLE INSTANCES OR WE CAN CREATE MULTIOLW INSTANCE IN MULRIPLE OF AVAILABILITY ZONE . BLOW IN FIGIRE . This decision is in maker or owner .

## Subnet-Level IP Reservation (The 5 Reserved IPs):

AWS reserves 5 IP addresses per subnet, not globally across the raw VPC CIDR. In every subnet created, AWS reserves:

.0: Network address

.1: VPC internal router

.2: Amazon-provided DNS (Route 53 Resolver)

.3: Reserved by AWS for future use

.255 (or the last IP in the block): Network broadcast address (AWS does not support broadcast, but the address remains reserved).


    whatever we crreate the resource 


WE DEPLOY OR WEBITE IN SAME REGION IN MULTIPLE SERVER ,  if crash happens in one server , so the website should still perform from other server as backup . or for load . we cann learn hee more about loadblancer . more details

![alt text](image-2.png)


    ![Region diagram showing three subnet columns labeled Subnet and Instances, arranged across a large rounded rectangle representing a region. Each subnet is placed in a separate Availability Zone, labeled a, b, and c, with a server stack beneath each zone. The diagram illustrates multiple EC2 instances spread across availability zones for redundancy, high availability, and load distribution. The wider environment is a clean, technical AWS architecture illustration on a light gray background with blue dashed lines separating the zones. Text in the image includes Region, Subnet, Instances, Availability Zone, a, b, and c. The tone is neutral and informative.](image-1.png)




![alt text](image-3.png)

![alt text](image-4.png)

 # ROUTUNG TABLE 
    A route table is a set of rules , called routes , that are used to determine where network traffic from our subnets or gateway is directed . each subnet in our VPC must be associated wit a route table  , whic controls the routing for that subnet.

    VPC ---> Subnet------> Routetable ------> Internet gateway 

![alt text](image-5.png)
 ![alt text](image-6.png) 
     how and from where to where is data is sent . look closely booth photos .



there is default VPC whic provided and manged by aws through which  we perform or all task.

  ![alt text](image-7.png)



# INTERNET GATEWAY 

            An internet Gateway is a compaonent  that allows  communication between instances in in your VPC ans the  interenet (world).


![alt text](image-8.png)




            Security Rules :- Network firewall rukes that  control inbound traffoc for instances.

            which data should flow in and out is defined by rules of security groups


    NOTE :- THis is a  firewall but it is a instances speific .
    for ex we can make diffrent diffrent security groups for diffrent Instances (EC2)

    or we wcan make one group and we share too 

    Internet gateway is attached to VPC




   # Network ACLs (Access control Lists):
          it works on subnet level

    Optional layer of security four our VPC that acts as firewall for cintrilling trraffic in and out of one or more subnets
   allow or dent rule.
     NOte :- in Security Groups we can only inbound and outbound means only Alloe . but in ACLs we and deny and allow both .

     NAT (Network address translation ) Gateway :- 
     Enables  instances in private subnet to connect to the internet or other AWS services , but orevents the internet from initiating connections to thise instances.

 ![alt text](image-9.png)

    ex- we made aprivate instance which is connected to databse 
        and the database doesnt need the connection from the internet 
        and we have to update our database  the aws has to download the files 
        . means we request traffic from inside the instance but not from the  outside  (dataleak) . one way communication
        so we use NAT 

        NAT can be used in public and private.

            Correction: A NAT Gateway must always reside in a public subnet and must have an allocated Elastic IP (EIP). The instances it serves sit in the private subnet. The private subnet's route table routes outbound internet traffic (0.0.0.0/0) to the NAT Gateway in the public subnet, which then forwards it to the Internet Gateway.

        

##    VPC PEERING:
    A networking connection between two VPC that enables you to route traffic  between them privately.
           
           if we have to coonec to vpc or more on doffrent region or samaae region or we have to enble/setup vpc in diffrent region 

           how we can syn the two vpc to get better result 
           so we use VPC peering 

    ![alt text](image-10.png)


## VPC endpoints :- 
    Allows you to privately  connect your VPC to suppported AWS serviices and VPC endpoints services powered by AWS Private link

![alt text](image-11.png)


## Bation Hosts :- 
 A special-purpose instances that provides      secure acces to your innstances inn private subnets.
like jump server

 ![alt text](image-12.png)

##     ELASTIC IP ADDRESSES : -
            Static IP address designed for dynamic compunting.

            whever we start or stop or instances , always a new IP is genrated 
            and or instance is connected to other instance or devices and other devices are reliable on other instances ip . If the IP keeps changing then how will the other devices get connected . so we need here static IP . that why we use ELASTIC IP ADDRESSESS.

* FOR MORE DETAIL ON ELASTIC IP FOLLOW THE PAGE :

    Please follow the [aws elastic](setup-guide.md) instructions carefully.
[Aws elastic ](<AWS elastic  IP.md>)

VPC Flow Logs: Capture information about the IP traffic  going to and from network interfaces in our VPC.

  ##                    DIRECT CONNECˇ:-----

 Establishing a dedicated network connection from our premises to AWS 

         lots of orgainzation are relying on cloud platforms
         so we have to remotely acces and work 
         we have to connect and in security point of view , we cannot make it public to expose on the internet.

         So here we establish a DIRECT CONNECT to get the connection privately , specially used in offices .

![alt text](image-13.png)

         AWS client VPN:
          Managed VPN service that enables remote access to AWS resources and on -premises networks using OpenVPN-based clients.


          ==========================================================


# Essential Architectural Rules to Add
##    Scope: VPC vs. Subnet Boundaries

A VPC spans an entire AWS Region across all its Availability Zones (AZs).

A Subnet can only exist within one single Availability Zone. It can never span across multiple AZs.

Stateful vs. Stateless Firewalls (Security Groups vs. NACLs)

Security Groups are Stateful: If an inbound request is permitted on port 443, the outbound response traffic is automatically allowed regardless of outbound rules.

Network ACLs (NACLs) are Stateless: Return traffic is not tracked. You must explicitly configure both Inbound and Outbound rules (including ephemeral ports 1024–65535 for return traffic). NACL rules are evaluated in numerical order (lowest number takes priority).

VPC Peering Non-Transitive Rule

VPC peering is strictly non-transitive. If VPC A is peered with VPC B, and VPC B is peered with VPC C, VPC A cannot communicate with VPC C through VPC B. You must establish a direct peering connection between A and C, or transition to an AWS Transit Gateway.

CIDR blocks of peered VPCs must never overlap.

## Two Types of VPC Endpoints

Gateway Endpoints: Free route-table targets used exclusively for Amazon S3 and Amazon DynamoDB.

Interface Endpoints (AWS PrivateLink): Deploys an Elastic Network Interface (ENI) with a private IP inside your subnet; supports most other AWS services and third-party SaaS for an hourly and data-processing fee.



# AWS VPC Routing Patterns & Core Rules

## 1. Route Table Configurations

This quick reference table contrasts public and private subnet route configurations:

| Subnet Type | Destination CIDR | Target | Purpose |
| :--- | :--- | :--- | :--- |
| **Public** | `10.0.0.0/16` | `local` | Internal communication across the VPC. |
| **Public** | `0.0.0.0/0` | `igw-xxxxxxxx` (Internet Gateway) | Direct, two-way internet access. |
| **Private** | `10.0.0.0/16` | `local` | Internal communication across the VPC. |
| **Private** | `0.0.0.0/0` | `nat-xxxxxxxx` (NAT Gateway) | Outbound-only internet access (e.g., for software updates or patches). |

---

## 2. Core Learning Rules (For Memorization)

To memorize this faster for exams or practical architecture, remember these four rules of thumb:

* **The `local` route is automatic:** Every route table inherently includes the local VPC CIDR (in this example, `10.0.0.0/16`) pointing to `local`. This ensures all subnets within the VPC can talk to each other by default.
* **`0.0.0.0/0` means "The Internet":** In networking, `0.0.0.0/0` translates to "any IP address not explicitly defined elsewhere." 
* **IGW defines a Public Subnet:** If you see `0.0.0.0/0` routed to an `igw-` (Internet Gateway), the subnet is public. It allows inbound and outbound traffic.
* **NAT defines a Private Subnet:** If you see `0.0.0.0/0` routed to a `nat-` (NAT Gateway), the subnet is private. It allows instances to reach the internet for updates, but blocks the internet from reaching inside.



#  THE BEST POSSIBLE WAY TO MEMORIZE THIS


##  Traffic Flow Diagrams



```text
Flow 1: Public Subnet to Internet
💻 EC2 Instance (Public Subnet)
└── 📤 Outbound Request (Destination: 0.0.0.0/0)
    └── 🗺️ Public Route Table
        └── 🚪 Internet Gateway (IGW)
            └── 🌐 The Internet

Flow 2: Private Subnet to Internet (Updates/Patches)
💻 EC2 Instance (Private Subnet)
└── 📤 Outbound Request (Destination: 0.0.0.0/0)
    └── 🗺️ Private Route Table
        └── 🔄 NAT Gateway (Deployed inside Public Subnet)
            └── 🗺️ Public Route Table
                └── 🚪 Internet Gateway (IGW)
                    └── 🌐 The Internet

Flow 3: Internal VPC Communication
💻 EC2 Instance A (Subnet A)
└── 📤 Internal Request (Destination: 10.0.x.x)
    └── 🗺️ Route Table (Target: local)
        └── 💻 EC2 Instance B (Subnet B)


```

# image form to understand
![alt text](<VPC .png>)

**1. The Outer Boundary (Region & VPC)**
The outermost layer of the architecture is the AWS Region, such as `us-east-1`, which dictates the geographical location of your resources. Inside this region sits your Virtual Private Cloud (VPC), assigned a master CIDR block of `10.0.0.0/16`. This VPC acts as your entirely isolated network perimeter within AWS.

**2. Fault Tolerance (Availability Zones)**
To ensure high availability, the VPC spans multiple physical data centers, represented here as Availability Zone 1 (`us-east-1a`) and Availability Zone 2 (`us-east-1b`). If hardware fails in AZ1, the redundant infrastructure in AZ2 keeps your web application running.

**3. Network Segmentation (Subnets)**
The primary `10.0.0.0/16` CIDR block is carved into smaller, specialized subnets within each Availability Zone.

* **Public Subnets (e.g., `10.0.1.0/24`):** These host resources that must face the internet directly, like your EC2 Web Servers.


* **Private Subnets (e.g., `10.0.2.0/24`):** These isolate sensitive resources, like backend EC2 App Servers and Amazon RDS databases, completely shielding them from direct external access.



**4. The Network Gateways (IGW & NAT)**
Traffic needs physical doorways to enter or exit the VPC.

* **Internet Gateway (IGW):** This component attaches directly to the VPC boundary. It provides the two-way street allowing public internet traffic to reach your public subnets and vice versa.


* **NAT Gateway:** Deployed strictly inside the Public Subnet and attached to a static Elastic IP address. It acts as a one-way mirror for your private instances, allowing them to initiate outbound connections (like downloading software patches) while blocking any incoming internet requests.



**5. The Traffic Navigators (Route Tables)**
Route Tables act as the GPS for your subnets, determining exactly where network packets should go.

* **Public Route Table:** Contains a rule routing `0.0.0.0/0` (all outside traffic) directly to the Internet Gateway (`igw-xxxx`).


* **Private Route Table:** Contains a rule routing `0.0.0.0/0` to the local NAT Gateway (`nat-yyyy`).


* **Local Route:** Both tables automatically include a rule pointing `10.0.0.0/16` to `local`, which allows the Web Servers in the public subnet to communicate internally with the App Servers in the private subnet.


## Network Access Control List (NACL)
**6. The Firewalls (NACLs & Security Groups)**
Security is applied at two distinct layers to control inbound and outbound traffic.

* **Network ACLs (NACLs):** These sit at the subnet boundary. They are stateless firewalls that can explicitly allow or deny traffic from entering or leaving an entire subnet.


* **Security Groups (SGs):** These sit directly on the Elastic network interface (ENI) of individual EC2 instances. They act as stateful firewalls. For example, the Web Server SG allows inbound HTTP traffic from the public, while the App Server SG strictly allows inbound traffic only from the Web Server's specific internal IP.

* Please follow the for more detail on security groups instructions carefully.

[security groups](Security_Groups.md)

When a user accesses your application, their request hits the public internet, flows through the IGW, passes the NACL rules, clears the Web Server's Security Group, and finally reaches the EC2 instance in the public subnet. If that Web Server needs database records, it initiates an internal request that travels seamlessly over the local route directly to the private subnet.