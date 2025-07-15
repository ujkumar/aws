# AWS VPC Setup: 2 Public and 2 Private Subnets

This guide walks you through setting up an AWS VPC with **2 public subnets** and **2 private subnets**, suitable for highly available web applications.

---

## ✅ Architecture Overview

| Component        | Quantity | Description                                          |
|------------------|----------|------------------------------------------------------|
| VPC              | 1        | Custom VPC to hold all subnets and resources         |
| Public Subnets   | 2        | One in each Availability Zone (AZ)                   |
| Private Subnets  | 2        | One in each AZ                                       |
| Internet Gateway | 1        | Allows internet access for public subnets            |
| NAT Gateway      | 1–2      | Allows private subnets to reach internet (e.g., updates) |
| Route Tables     | 2        | One for public, one for private subnets              |

---

## 🚀 Step-by-Step Instructions (AWS Console)

### Step 1: Create a Custom VPC

1. Navigate to **VPC Dashboard > VPCs > Create VPC**
2. Set:
   - **Name**: `MyCustomVPC`
   - **IPv4 CIDR**: `10.0.0.0/16`
   - **Tenancy**: Default
3. Click **Create VPC**

---

### Step 2: Create Subnets

You’ll create 4 subnets across 2 Availability Zones (e.g., `us-east-1a` and `us-east-1b`):

#### Public Subnets

| Name           | AZ         | CIDR Block   |
|----------------|------------|--------------|
| PublicSubnet1  | us-east-1a | 10.0.1.0/24  |
| PublicSubnet2  | us-east-1b | 10.0.2.0/24  |

#### Private Subnets

| Name           | AZ         | CIDR Block   |
|----------------|------------|--------------|
| PrivateSubnet1 | us-east-1a | 10.0.3.0/24  |
| PrivateSubnet2 | us-east-1b | 10.0.4.0/24  |

Steps:

1. Go to **Subnets > Create Subnet**
2. Select your VPC: `MyCustomVPC`
3. Enter Name, AZ, and CIDR block
4. Repeat for all 4 subnets

---

### Step 3: Create and Attach Internet Gateway

1. Go to **Internet Gateways > Create Internet Gateway**
   - Name: `MyInternetGateway`
2. Create and **attach** it to `MyCustomVPC`

---

### Step 4: Create Route Tables

#### Public Route Table

1. Go to **Route Tables > Create Route Table**
   - Name: `PublicRouteTable`
   - VPC: `MyCustomVPC`
2. In **Routes > Edit Routes**, add:
   - `0.0.0.0/0` → Target: **Internet Gateway**
3. Associate with **PublicSubnet1** and **PublicSubnet2**

#### Private Route Table

1. Create a second route table:
   - Name: `PrivateRouteTable`
   - VPC: `MyCustomVPC`
2. No route yet
3. Associate with **PrivateSubnet1** and **PrivateSubnet2**

---

### Step 5: Create NAT Gateway

1. Go to **Elastic IPs > Allocate Elastic IP**
2. Go to **NAT Gateways > Create NAT Gateway**
   - Subnet: **PublicSubnet1**
   - Elastic IP: The one you just allocated
   - Name: `MyNatGateway`

---

### Step 6: Update Private Route Table

1. Go to **Route Tables > PrivateRouteTable > Edit Routes**
2. Add route:
   - `0.0.0.0/0` → Target: **NAT Gateway**

---

## ✅ Final Architecture Diagram

```plaintext
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
