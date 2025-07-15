Setting up a VPC (Virtual Private Cloud) with 2 private subnets and 2 public subnets is a foundational networking task in AWS. This type of architecture is commonly used for deploying high-availability web applications where:

Public subnets host resources like load balancers, NAT gateways, or public-facing EC2 instances.

Private subnets host databases, application servers, and other resources that shouldn't be directly accessed from the internet.

✅ Overview of What You'll Set Up:
Component	Quantity	Description
VPC	1	Custom VPC to hold all subnets and resources
Public Subnets	2	One in each Availability Zone (AZ)
Private Subnets	2	One in each AZ
Internet Gateway	1	Allows internet access for public subnets
NAT Gateway	1–2	Allows private subnets to reach internet (e.g., for updates)
Route Tables	2	One for public, one for private subnets

🚀 Step-by-Step Setup in AWS Console
Step 1: Create a Custom VPC
Go to VPC Dashboard > VPCs > Create VPC

Choose:

Name tag: MyCustomVPC

IPv4 CIDR block: 10.0.0.0/16 (provides 65,536 IPs)

Leave IPv6 (optional)

Tenancy: Default

Click Create VPC

Step 2: Create Subnets
You’ll need 4 subnets, distributed across 2 AZs (e.g., us-east-1a and us-east-1b):

✅ Public Subnets:
Name	AZ	CIDR Block
PublicSubnet1	us-east-1a	10.0.1.0/24
PublicSubnet2	us-east-1b	10.0.2.0/24

✅ Private Subnets:
Name	AZ	CIDR Block
PrivateSubnet1	us-east-1a	10.0.3.0/24
PrivateSubnet2	us-east-1b	10.0.4.0/24

To create:

Go to Subnets > Create Subnet

Select your VPC (MyCustomVPC)

Enter Name, AZ, and CIDR block as above

Repeat for all 4 subnets

Step 3: Create and Attach Internet Gateway
Go to Internet Gateways > Create Internet Gateway

Name: MyInternetGateway

Create, then attach it to MyCustomVPC

Step 4: Create Route Tables
✅ Public Route Table
Go to Route Tables > Create Route Table

Name: PublicRouteTable

VPC: MyCustomVPC

After creation, go to Routes > Edit routes

Add route: 0.0.0.0/0 → Target: Internet Gateway

Go to Subnet Associations > Edit subnet associations

Associate PublicSubnet1 and PublicSubnet2

✅ Private Route Table
Create another route table:

Name: PrivateRouteTable

VPC: MyCustomVPC

Don’t add a route yet (we’ll do it after setting up NAT)

Associate with PrivateSubnet1 and PrivateSubnet2

Step 5: Create NAT Gateway
You need an Elastic IP (EIP) first:

Go to Elastic IPs > Allocate Elastic IP address

Allocate and note the EIP

Then:

Go to NAT Gateways > Create NAT Gateway

Subnet: Choose PublicSubnet1

Elastic IP: Use the allocated one

Name: MyNatGateway

Click Create NAT Gateway

⚠️ NAT Gateway must be in a public subnet to work!

Step 6: Update Private Route Table
Go to Route Tables > PrivateRouteTable > Routes > Edit routes

Add route: 0.0.0.0/0 → Target: NAT Gateway

✅ Final Diagram Summary
plaintext
Copy
Edit
          INTERNET
              |
       [Internet Gateway]
              |
   ----------------------------
   |                          |
[PublicSubnet1]        [PublicSubnet2]
 (NAT Gateway)             (optional)
     |                         |
     |                   Load Balancer (optional)
     |                         |
----------------  ----------------
|              |  |              |
PrivateSubnet1   PrivateSubnet2
(App/DB Servers) (App/DB Servers)
✅ Optional Enhancements
High Availability NAT: Add a 2nd NAT Gateway in PublicSubnet2 and update routing.

Security Groups & NACLs: Use for traffic control.

Bastion Host: Deploy in public subnet to SSH into private resources.