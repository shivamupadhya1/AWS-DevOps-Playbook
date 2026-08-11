# NAT Gateway

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 13
>
> NAT Gateway

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand why a NAT Gateway is needed
- Explain Source NAT (SNAT)
- Understand packet flow
- Explain why NAT Gateway must be in a public subnet
- Differentiate NAT Gateway and NAT Instance
- Design production architectures
- Troubleshoot internet connectivity issues from private subnets
- Answer interview questions confidently

---

# 1. What is a NAT Gateway?

A NAT (Network Address Translation) Gateway is an AWS-managed service that allows **instances in private subnets to access the internet**, while preventing the internet from initiating connections to those instances.

Think of it as a **one-way door**:

```
Private EC2  ─────► Internet

Internet  ─────X────► Private EC2
```

The private instance can initiate outbound communication, but unsolicited inbound traffic from the internet is blocked.

---

# 2. Why Do We Need a NAT Gateway?

Imagine you have:

- Application Server
- Database
- Kubernetes Worker Node

All are in a private subnet.

They still need to:

- Download OS updates
- Pull Docker images
- Access Amazon S3
- Access Secrets Manager
- Install packages
- Call third-party APIs

Without a NAT Gateway:

```
Private EC2

↓

No Route to Internet

↓

Package Installation Fails
```

---

# 3. Why Not Give Every EC2 a Public IP?

Because production workloads should remain private.

Giving backend servers public IPs:

- Increases attack surface
- Violates security best practices
- Exposes services unnecessarily

Instead:

```
Internet

↓

NAT Gateway

↓

Private EC2
```

---

# 4. NAT Gateway Architecture

```
                   Internet
                       │
                Internet Gateway
                       │
                Public Subnet
                       │
                 NAT Gateway
                       │
              Private Route Table
                       │
              Private Subnet
                       │
                  Application EC2
```

---

# 5. Why Must NAT Gateway Be in a Public Subnet?

A NAT Gateway needs internet access itself.

Therefore:

- Public subnet
- Internet Gateway attached
- Elastic IP associated

Without these, it cannot forward traffic.

---

# 6. Source NAT (SNAT)

NAT Gateway performs **Source Network Address Translation (SNAT)**.

Example

Private EC2

```
10.0.2.15
```

Requests

```
https://google.com
```

NAT Gateway changes

```
Source IP

10.0.2.15

↓

54.xx.xx.xx (Elastic IP)
```

Google sees only the Elastic IP of the NAT Gateway.

---

# 7. Packet Flow

```
Private EC2

↓

Security Group

↓

Private Route Table

↓

NAT Gateway

↓

Internet Gateway

↓

Internet
```

Response

```
Internet

↓

Internet Gateway

↓

NAT Gateway

↓

Private EC2
```

---

# 8. Route Table Configuration

Private Route Table

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | NAT Gateway |

This tells private instances to send all internet-bound traffic to the NAT Gateway.

---

# 9. Public Route Table

The public subnet hosting the NAT Gateway requires:

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

---

# 10. Elastic IP Requirement

A NAT Gateway must have an Elastic IP.

Reason:

The internet cannot route traffic back to a private IP.

The Elastic IP acts as the public identity of the NAT Gateway.

---

# 11. NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|----------|-------------|--------------|
| Managed by AWS | ✅ | ❌ |
| Auto Scaling | AWS Managed | Manual |
| High Availability | AWS Managed (within an AZ) | Manual |
| Maintenance | None | Customer Responsible |
| Performance | High | Depends on EC2 |
| Security Patching | AWS | Customer |
| Elastic IP | Required | Required |

AWS recommends using NAT Gateway for most production workloads.

---

# 12. High Availability Design

Bad Design

```
AZ-A

NAT Gateway

↓

AZ-B Private EC2
```

If AZ-A fails, AZ-B loses outbound internet access.

---

Recommended Design

```
AZ-A

Public Subnet

↓

NAT Gateway-A

↓

Private EC2-A

------------------------

AZ-B

Public Subnet

↓

NAT Gateway-B

↓

Private EC2-B
```

Each Availability Zone has its own NAT Gateway.

---

# 13. Cost Considerations

NAT Gateway charges for:

- Hourly usage
- Data processed

If most traffic is to Amazon S3 or DynamoDB, consider using:

- Gateway VPC Endpoint

This can reduce NAT Gateway costs.

---

# 14. Common Use Cases

- Private EC2 package updates
- Kubernetes worker nodes
- ECS tasks
- Docker image downloads
- AWS API calls
- Third-party REST APIs

---

# 15. Production Architecture

```
Internet
     │
Internet Gateway
     │
Public Subnet
     │
ALB
     │
NAT Gateway
     │
Private Subnet
     │
Application
     │
Database
```

Only the ALB is internet-facing.

Application servers remain private.

---

# 16. Production Scenarios

## Scenario 1

### Problem

Private EC2 cannot install packages.

Investigation

- NAT Gateway exists?
- Route Table points to NAT Gateway?
- NAT Gateway status is Available?
- Internet Gateway attached?
- Security Group outbound rules?
- NACL outbound rules?
- DNS resolution?

---

## Scenario 2

### Problem

Private EC2 cannot pull Docker images.

Check

- NAT Gateway
- Route Table
- DNS
- Proxy settings
- Docker daemon configuration

---

## Scenario 3

### Problem

Private EKS Nodes cannot join the cluster.

Possible causes

- NAT Gateway missing
- Route Table incorrect
- IAM Role issue
- Security Group
- DNS
- Cluster endpoint accessibility

---

## Scenario 4

### Problem

Private EC2 cannot access Amazon S3.

Check

- NAT Gateway
- Route Table
- IAM Role
- Bucket Policy
- Gateway Endpoint (if configured)

---

## Scenario 5

### Problem

NAT Gateway is healthy but traffic still fails.

Check

- Private Route Table
- NACL
- Security Groups
- DNS
- Operating system firewall

---

# 17. Best Practices

- Deploy one NAT Gateway per Availability Zone.
- Use private subnets for backend workloads.
- Use VPC Endpoints for S3 and DynamoDB where possible.
- Monitor NAT Gateway metrics in CloudWatch.
- Avoid routing cross-AZ traffic through a single NAT Gateway.

---

# 18. Common Mistakes

❌ Creating NAT Gateway in a private subnet.

❌ Forgetting the Elastic IP.

❌ Missing `0.0.0.0/0` route in private Route Table.

❌ Assuming NAT Gateway allows inbound internet traffic.

❌ Using a single NAT Gateway for all Availability Zones without understanding availability and cost implications.

---

# 19. Troubleshooting Flow

Private EC2 has no internet.

```
Step 1

Private Route Table

↓

Step 2

Target = NAT Gateway?

↓

Step 3

NAT Gateway Available?

↓

Step 4

Elastic IP Attached?

↓

Step 5

Internet Gateway Attached?

↓

Step 6

Security Group

↓

Step 7

NACL

↓

Step 8

DNS

↓

Step 9

Application
```

---

# 20. Interview Questions

## Question 1

What is a NAT Gateway?

### Answer

A NAT Gateway is an AWS-managed service that enables instances in private subnets to initiate outbound internet connections while preventing unsolicited inbound connections from the internet.

---

## Question 2

Why must a NAT Gateway be placed in a public subnet?

### Answer

Because it requires internet connectivity through an Internet Gateway and uses an Elastic IP to communicate with external networks.

---

## Question 3

Does NAT Gateway allow inbound internet traffic?

### Answer

No.

Only traffic that is part of an outbound connection initiated by a private instance is allowed back.

---

## Question 4

Why is an Elastic IP required?

### Answer

The NAT Gateway needs a public address so internet hosts can send response traffic back. The Elastic IP provides this stable public identity.

---

## Question 5

What is Source NAT?

### Answer

Source NAT replaces the source IP address of outbound packets. A private IP (for example, `10.0.2.15`) is translated to the NAT Gateway's Elastic IP before the traffic leaves the VPC.

---

## Question 6

NAT Gateway vs Internet Gateway?

### Answer

Internet Gateway provides internet connectivity for resources with public IPs. NAT Gateway provides outbound-only internet connectivity for private resources.

---

## Question 7

Can a NAT Gateway exist without an Internet Gateway?

### Answer

No.

Without an Internet Gateway, the NAT Gateway cannot communicate with the public internet.

---

## Question 8

How many NAT Gateways should be deployed in production?

### Answer

Ideally, one per Availability Zone. This avoids a single point of failure and prevents cross-AZ routing if one AZ becomes unavailable.

---

# 21. Amazon Follow-up Questions

### Question

Can a NAT Gateway receive SSH connections?

### Answer

No.

It is not a general-purpose server and does not accept inbound administrative connections.

---

### Question

Can a NAT Gateway connect two VPCs?

### Answer

No.

For VPC-to-VPC communication, use VPC Peering or Transit Gateway.

---

### Question

Can private instances communicate with each other through a NAT Gateway?

### Answer

No.

Traffic within the same VPC uses the local route and does not traverse the NAT Gateway.

---

### Question

Can Lambda functions in a private subnet use a NAT Gateway?

### Answer

Yes.

If a Lambda function is attached to a private subnet and needs internet access, its subnet Route Table can direct outbound traffic to a NAT Gateway.

---

# 22. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Create:

- VPC
- Public Subnet
- Private Subnet

---

## Lab 2

Create an Internet Gateway.

Attach it to the VPC.

---

## Lab 3

Create a NAT Gateway in the public subnet.

Associate an Elastic IP.

---

## Lab 4

Update the private Route Table:

```
0.0.0.0/0

↓

NAT Gateway
```

---

## Lab 5

Launch:

- Public EC2
- Private EC2

Verify:

- Public EC2 has direct internet access.
- Private EC2 accesses the internet only through the NAT Gateway.

---

## Lab 6

Stop or remove the NAT Gateway.

Observe:

- Private EC2 loses outbound internet connectivity.
- Public EC2 continues to function normally.

---

# 23. One-Page Revision

```
Private EC2
      │
Security Group
      │
Private Route Table
      │
NAT Gateway
      │
Elastic IP
      │
Internet Gateway
      │
Internet
```

Remember:

- NAT Gateway = Outbound internet for private subnets.
- Requires Elastic IP.
- Must be in a public subnet.
- Performs Source NAT.
- Does not accept unsolicited inbound internet traffic.
- One NAT Gateway per AZ is the recommended production design.

---

# Think Like a Production Engineer

When a private instance cannot access the internet, don't immediately blame the NAT Gateway.

Trace the entire path:

```
Private EC2
      │
Route Table
      │
NAT Gateway
      │
Elastic IP
      │
Internet Gateway
      │
Internet
```

At each hop, verify configuration before moving to the next. This systematic approach is how production incidents are investigated and resolved.

# End of Chapter
