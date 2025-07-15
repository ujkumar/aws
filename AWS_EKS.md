
# 🎛️ Create a Highly Available and Secure EKS Cluster via AWS Console

This guide walks you through creating a **highly available** and **secure** Amazon EKS cluster **entirely via the AWS Console**. We’ll cover setting up the network, EKS cluster, worker nodes, and best practices for security.

---

## ✅ Prerequisites

Before you start, make sure you have:

- AWS IAM user/role with admin privileges
- AWS Region selected (e.g., `us-east-1`)
- A key pair (or create one under **EC2 > Key Pairs**)
- IAM roles for EKS and EC2 (can be created during setup)

---

## 🔧 Step 1: Create a VPC with Public and Private Subnets

1. Go to **VPC Console > Launch VPC Wizard**
2. Choose **VPC with Public and Private Subnets**
3. Configure:
   - Name: `eks-vpc`
   - AZs: Choose **2 AZs**
   - 2 Public and 2 Private subnets (one in each AZ)
   - Enable **NAT Gateway** (for private subnets)
   - Enable **DNS hostnames**
4. Click **Create VPC**

This creates a **highly available** network with internet access for public and NAT-ed access for private subnets.

---

## ☸️ Step 2: Create EKS Cluster (Control Plane)

1. Go to **EKS Console > Clusters > Add Cluster > Create**
2. Enter:
   - Name: `my-eks-cluster`
   - Kubernetes version: Choose latest stable (e.g., `1.29`)
3. **Cluster Service Role**:
   - Create a new IAM role with `AmazonEKSClusterPolicy` attached
4. **Networking**:
   - Choose the VPC created in Step 1
   - Select **2 private subnets** (one per AZ) for **high availability**
   - Enable **Public Access** if you want to access the API server from outside VPC
     - Restrict CIDR (e.g., your IP)
5. **Secrets Encryption (Recommended)**:
   - Enable and choose/create KMS key
6. **Logging**:
   - Enable at least: `api`, `audit`, `authenticator`
7. Click **Create Cluster**

🕐 This may take 10–15 minutes.

---

## 👷 Step 3: Add Worker Nodes (Managed Node Group)

1. After the control plane is ACTIVE, go to **Compute > Add Node Group**
2. Configure:
   - Name: `worker-nodes`
   - IAM Role: Create a new one with `AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, and `AmazonEKS_CNI_Policy`
3. Networking:
   - Subnets: Select the **private subnets**
   - Remote access: Enable SSH if needed (attach key pair)
4. Scaling:
   - Min: 2 | Max: 5 | Desired: 2
5. Click **Create**

---

## 🛡️ Step 4: Enable IAM OIDC and IRSA

1. Go to **EKS > Your Cluster > Configuration > Authentication**
2. Click **Associate OIDC Provider**
   - This enables **IAM Roles for Service Accounts (IRSA)**
   - Accept default and click **Associate**

IRSA lets your pods access AWS services securely using IAM roles.

---

## 🌐 Step 5: Install ALB Ingress Controller (Optional)

1. Go to **IAM > Roles** and create an IAM role for the controller:
   - Attach the required policy (use AWS documentation or console wizard)
2. Create a Kubernetes service account mapped to the role
3. Deploy the controller via Helm or manifest from your local machine

> This enables public HTTP/S ingress via ALB.

---

## 📊 Step 6: Enable CloudWatch Logging

1. Go to **EKS > Your Cluster > Logging**
2. Enable control plane logging: `api`, `audit`, `authenticator`, `scheduler`
3. (Optional) Install CloudWatch agent on nodes for application logs

---

## 🧪 Step 7: Connect and Test Access

1. Install & configure AWS CLI and `kubectl` locally
2. Update kubeconfig:
   ```bash
   aws eks update-kubeconfig --name my-eks-cluster --region us-east-1
   ```
3. Test access to the cluster:
   ```bash
   kubectl get nodes
   ```

---

## 🔐 Security Best Practices

| Area                  | Best Practice                                               |
|-----------------------|-------------------------------------------------------------|
| Node Networking       | Use **private subnets** for all worker nodes                |
| API Access            | Restrict to specific IP CIDRs                               |
| Secrets               | Enable **KMS encryption** for secrets at rest               |
| IAM Roles             | Use **IRSA** for pod-level access to AWS                    |
| Security Groups       | Restrict inbound/egress rules tightly                       |
| Logging & Monitoring  | Enable control plane logs + install CloudWatch agent        |
| Ingress               | Use **ALB ingress controller** with HTTPS via ACM           |
| Upgrades              | Regularly upgrade Kubernetes version                        |

---

## 🧭 Summary Architecture

```plaintext
           [Internet]
                |
       [ALB - Public Subnets]
                |
          [EKS Control Plane]
                |
   ---------------------------------
   |                               |
[Private Subnet 1]         [Private Subnet 2]
     |                           |
 [Worker Nodes]            [Worker Nodes]
     |                           |
    [Pods]     <-->     [IRSA IAM Roles]
     |
 [RDS / S3 / Other AWS Services]
```

---

## 📚 References

- [Amazon EKS Console Docs](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)
- [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [EKS Best Practices Guide](https://aws.github.io/aws-eks-best-practices/)
