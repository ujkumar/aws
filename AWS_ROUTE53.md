
# 🌐 Creating Route 53 Hosted Zone & DNS Records via AWS Console

This guide explains how to use the **AWS Management Console** to create a **Route 53 hosted zone**, configure **DNS records**, and optionally use it for domain registration or internal DNS routing.

---

## ✅ Prerequisites

- An AWS account
- A registered domain (via AWS or external registrar)
- Access to the AWS Console

---

## 📦 Step 1: Navigate to Route 53

1. Go to the **AWS Console**.
2. In the **Services** menu, search for and open **Route 53**.

---

## 🧭 Step 2: Choose Your DNS Type

There are two types of hosted zones:

| Type               | Use Case                                           |
|--------------------|----------------------------------------------------|
| Public Hosted Zone | For internet-facing domains (e.g. `example.com`)  |
| Private Hosted Zone| For internal DNS within a VPC                     |

---

## 🌍 Step 3: Create a Hosted Zone

1. In the Route 53 dashboard, go to **Hosted Zones** > **Create hosted zone**
2. Fill in:
   - **Domain name**: `example.com`
   - **Type**:
     - Select **Public hosted zone** for internet-accessible domains
     - Select **Private hosted zone** for VPC-internal resolution
   - (Optional) Add a comment
   - If **Private**, associate with one or more **VPCs**
3. Click **Create hosted zone**

📌 Route 53 will automatically create two default records:
- **NS** (Name Server) record
- **SOA** (Start of Authority) record

---

## ✏️ Step 4: Add DNS Records (A, CNAME, etc.)

1. Go into your hosted zone
2. Click **Create record**
3. Configure the record:
   - **Record name**: `www` (for `www.example.com`)
   - **Record type**:
     - `A` for IP address
     - `CNAME` for alias (e.g., `www → example.com`)
     - `MX`, `TXT`, `SRV` as needed
   - **Value**:
     - For `A` record: IP address or alias to AWS service
     - For `CNAME`: the domain to point to
   - **Routing Policy**:
     - Simple (default)
     - Weighted, Latency, Failover, Geo for advanced use
   - **TTL (Time to Live)**: 300 seconds (default)

4. Click **Create records**

---

## 🌐 Step 5: Update Domain Registrar (If Domain Registered Outside AWS)

If you bought your domain from a **third-party registrar** (e.g., GoDaddy, Namecheap):

1. Go to Route 53 > Your Hosted Zone
2. Copy the **Name Server (NS) records**
3. Log into your domain registrar
4. Replace their default name servers with the ones provided by Route 53

📌 It may take a few hours for the DNS changes to propagate.

---

## 🔒 Step 6: Enable DNSSEC (Optional)

1. In the hosted zone, go to **DNSSEC signing**
2. Click **Enable DNSSEC signing**
3. Follow steps to add key-signing key (KSK)
4. Register DNSSEC info with your domain registrar

---

## 🧪 Step 7: Test Your DNS Records

You can test DNS resolution using:

```bash
nslookup www.example.com
dig www.example.com
```

Or use online tools like:
- https://dnschecker.org
- https://toolbox.googleapps.com/apps/dig

---

## 🧭 Summary Architecture

```plaintext
        [User Browser]
              |
         DNS Lookup
              |
    +------------------------+
    |  Route 53 Hosted Zone  |
    +------------------------+
     |         |         |
   A Record   CNAME     MX/TXT
     |         |          |
    EC2     ALB/S3    SES/Email
```

---

## 📌 Notes

- TTL controls DNS caching duration; lower TTL means faster updates.
- Alias records are recommended for AWS resources (e.g., ALB, CloudFront).
- Use **Failover Routing + Health Checks** for high availability.
- For internal domains, use **Private Hosted Zones** with VPCs.
- Combine with **ACM and ALB** for HTTPS applications.
- DNSSEC adds authenticity to DNS responses, helping to prevent spoofing.

---

## 📚 References

- [Route 53 Documentation](https://docs.aws.amazon.com/route53/)
- [Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- [DNSSEC with Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring-dnssec.html)
