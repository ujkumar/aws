# 🖥️ AWS EC2 Instance Provisioning in Public and Private Subnets

This guide walks you through provisioning EC2 instances in both **public** and **private subnets** within a custom VPC using the **AWS EC2 Console**.

---

## 🧱 Prerequisites

Before provisioning EC2 instances, make sure you have the following set up:

- A **VPC** with:
  - 2 public subnets
  - 2 private subnets
  - Internet Gateway (attached to VPC)
  - NAT Gateway (in a public subnet)
  - Proper route tables
- At least one **key pair** to SSH into instances
- Security groups configured for desired access

---

## ✅ Step-by-Step: Launch EC2 in Public Subnet

### 1. Navigate to EC2 Console

- Go to **EC2 > Instances > Launch Instance**

### 2. Configure Instance

- **Name**: `PublicEC2Instance`
- **AMI**: Amazon Linux 2 / Ubuntu (depending on preference)
- **Instance type**: t2.micro (Free tier eligible)
- **Key pair**: Select existing or create new one
- **Network settings**:
  - **VPC**: Choose your custom VPC
  - **Subnet**: Choose one of the **public subnets**
  - **Auto-assign public IP**: **Enable**
- **Firewall (Security Group)**:
  - Create or select a security group that allows:
    - SSH (port 22) from your IP
    - HTTP (port 80) or HTTPS (port 443) if hosting a web server

### 3. Storage, Tags, etc.

- Leave default storage unless you need more
- Add tags (optional but recommended)

### 4. Launch Instance

- Click **Launch Instance**
- Use the key pair to SSH into the instance once it’s running:
  ```bash
  ssh -i my-key.pem ec2-user@<public-ip>

---

## ✅ Step-by-Step: Launch EC2 in Private Subnet

### 1. Navigate to EC2 Console

- Go to **EC2 > Instances > Launch Instance**

### 2. Configure Instance

- **Name**: `PrivateEC2Instance`
- **AMI**: Amazon Linux 2 / Ubuntu (depending on preference)
- **Instance type**: t2.micro (Free tier eligible)
- **Key pair**: Select existing or create new one
- **Network settings**:
  - **VPC**: Choose your custom VPC
  - **Subnet**: Choose one of the **private subnets**
  - **Auto-assign public IP**: **Enable**
- **Firewall (Security Group)**:
  - Create or select a security group that allows:
    - SSH (port 22) from your IP
    - HTTP (port 80) or HTTPS (port 443) if hosting a web server

### 3. Storage, Tags, etc.

- Leave default storage unless you need more
- Add tags (optional but recommended)

### 4. Launch Instance

- Click **Launch Instance**
- Use the key pair to SSH into the instance once it’s running:
  ```bash
  ssh -i my-key.pem ec2-user@<private-ip>

---

# ⚙️ AWS Architecture: Load Balancer, Database Hosting, and Auto Scaling Groups

This guide explains how to:

- ✅ Set up **Load Balancers** in public subnets
- ✅ Host **Databases** in private subnets
- ✅ Use **Auto Scaling Groups (ASGs)** for scalable workloads in public/private subnets

These practices are key for building secure, highly available, and scalable architectures on AWS.

---

## ✅ 1. Set Up Load Balancer in Public Subnets

### What It Does:
- Distributes traffic across multiple EC2 instances
- Publicly accessible via a DNS name
- Helps achieve fault tolerance and high availability

### Steps to Create Application Load Balancer (ALB):

1. **Go to EC2 Console > Load Balancers > Create Load Balancer**
2. Choose **Application Load Balancer**
3. Set:
   - **Name**: `PublicALB`
   - **Scheme**: `internet-facing`
   - **IP address type**: IPv4
   - **Listeners**: Add listener on port 80 (HTTP) or 443 (HTTPS)
4. **Availability Zones**:
   - Select your **VPC**
   - Choose both **public subnets**
5. **Configure Security Group**:
   - Allow inbound access on ports 80/443
6. **Target Group**:
   - Create a new target group
   - Target type: `instance` or `IP` (use `instance` for EC2)
   - Register instances (e.g., in private subnets)
7. **Review and Create**

> Now your ALB is reachable via its DNS and routes traffic to EC2 instances in private subnets.

---

## ✅ 2. Host Databases in Private Subnets

### Why Use Private Subnets:
- Databases shouldn't be exposed to the internet
- Enhances security by restricting access

### Hosting an RDS Database:

1. **Go to RDS Console > Create Database**
2. Choose:
   - Engine: Amazon Aurora / MySQL / PostgreSQL
   - Deployment: Standard create
3. **Settings**:
   - DB instance identifier: `my-db`
   - Credentials: Create master username/password
4. **Connectivity**:
   - VPC: Choose your custom VPC
   - Subnet group: Select **private subnets**
   - Public access: **No**
   - VPC security group: Ensure access from EC2 app servers
5. **Database options**: Optional
6. **Monitoring, backup, maintenance**: Configure as needed
7. Click **Create database**

> The database will only be accessible from inside the VPC (typically from private EC2 instances).

---

## ✅ 3. Use Auto Scaling Groups (ASG)

### What It Does:
- Automatically adjusts EC2 capacity based on traffic/load
- Ensures high availability and fault tolerance
- Works well with Load Balancers

### Steps to Set Up ASG:

1. **Go to EC2 Console > Auto Scaling Groups > Create Auto Scaling Group**
2. **Name**: `MyASG`
3. **Launch Template/Configuration**:
   - Use an existing **launch template**
     - AMI, instance type, key pair, security group
     - Select public/private subnet depending on the use case
4. **Network Settings**:
   - VPC: Choose your custom VPC
   - Subnets:
     - Use **public subnets** for front-end/web apps
     - Use **private subnets** for app servers
5. **Attach Load Balancer**:
   - Attach existing ALB (optional but recommended)
6. **Set Desired, Min, Max Capacity**:
   - e.g., Min: 2, Desired: 2, Max: 6
7. **Scaling Policies**:
   - Use Target tracking scaling
   - Example: Maintain 60% average CPU utilization
8. **Review and Create**

> The ASG will automatically launch or terminate EC2 instances based on the policy.

---

## ✅ Summary Architecture

```plaintext
         [INTERNET]
             |
     [Application Load Balancer]
             |
      ---------------------
      |                   |
 [PublicSubnet1]    [PublicSubnet2]
      |                   |
  [Auto Scaling Group - Web Tier]
      |
  ---------------------
  |                   |
[PrivateSubnet1]  [PrivateSubnet2]
      |
   [RDS Database]
