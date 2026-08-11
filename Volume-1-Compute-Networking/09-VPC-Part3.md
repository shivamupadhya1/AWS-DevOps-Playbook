# Amazon Virtual Private Cloud (VPC)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 09
>
> Part 3 – Security, Connectivity & Production Troubleshooting

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand Security Groups
- Understand Network ACLs
- Differentiate Stateful vs Stateless firewalls
- Understand VPC Peering
- Understand Transit Gateway
- Understand VPC Endpoints
- Understand AWS PrivateLink
- Use VPC Flow Logs
- Troubleshoot production networking issues
- Answer senior-level VPC interview questions

---

# 1. Security Group

A Security Group acts as a **virtual firewall** attached to an AWS resource.

Examples:

- EC2
- ALB
- RDS
- ECS
- EKS Nodes
- Lambda (inside VPC)

It controls traffic **to and from** the resource.

---

# 2. Security Group Architecture

```
Internet

↓

Internet Gateway

↓

Security Group

↓

EC2
```

The Security Group evaluates every packet before it reaches the instance.

---

# 3. Stateful Firewall

Security Groups are **stateful**.

Example

```
Laptop

↓

EC2 Port 22 Allowed

↓

SSH Connection Established

↓

Return Traffic

↓

Automatically Allowed
```

You only configure the inbound rule.

The response traffic is automatically permitted.

---

# 4. Security Group Rules

Two types of rules:

Inbound Rules

```
Who can reach me?
```

Outbound Rules

```
Where can I connect?
```

---

# 5. Example

Inbound

| Protocol | Port | Source |
|----------|------|---------|
| SSH | 22 | Your IP |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |

Outbound

```
Allow All
```

(Default configuration)

---

# 6. Best Practices for Security Groups

- Allow only required ports.
- Restrict SSH to trusted IPs.
- Never expose databases publicly.
- Use Security Group references instead of IPs where possible.
- Review rules periodically.

---

# 7. Network ACL (NACL)

A Network ACL is another firewall, but it operates at the **subnet level**.

It controls traffic entering and leaving the subnet.

```
Internet

↓

NACL

↓

Security Group

↓

EC2
```

---

# 8. Stateless Firewall

NACLs are **stateless**.

Example:

If inbound SSH is allowed:

```
Laptop

↓

Port 22

↓

EC2
```

You must also allow the return traffic (ephemeral ports) in the outbound rules.

Unlike Security Groups, NACLs do not automatically allow response traffic.

---

# 9. Security Group vs NACL

| Feature | Security Group | NACL |
|----------|----------------|------|
| Level | Instance/Resource | Subnet |
| Stateful | ✅ Yes | ❌ No |
| Allow Rules | ✅ Yes | ✅ Yes |
| Deny Rules | ❌ No | ✅ Yes |
| Evaluated | Resource | Subnet Boundary |

---

# 10. When Should You Use NACL?

Most workloads rely mainly on Security Groups.

Use NACLs when you need subnet-wide controls, such as:

- Blocking a malicious IP range.
- Compliance requirements.
- Additional defense-in-depth.

---

# 11. VPC Peering

By default, VPCs are isolated.

VPC Peering creates a private connection between two VPCs.

```
VPC-A

⇄

VPC-B
```

Traffic stays on the AWS backbone.

---

# 12. VPC Peering Rules

Requirements:

- Non-overlapping CIDR blocks.
- Route Tables updated in both VPCs.
- Security Groups allow traffic.

---

# 13. Limitations of VPC Peering

No transitive routing.

Example

```
VPC-A

↓

VPC-B

↓

VPC-C
```

If A is peered with B and B with C, A **cannot** automatically communicate with C.

---

# 14. Transit Gateway

Transit Gateway acts as a central network hub.

```
          Transit Gateway

      /         |          \

    VPC-A     VPC-B      VPC-C

         |

     On-Premises
```

Benefits:

- Simplified routing
- Centralized connectivity
- Supports hundreds or thousands of networks
- Better scalability than many VPC Peerings

---

# 15. VPC Endpoints

Sometimes an EC2 instance in a private subnet needs to access AWS services without using the internet.

VPC Endpoints provide this capability.

Example

Private EC2

↓

S3

↓

No NAT Gateway

↓

No Internet

Traffic remains inside AWS.

---

# 16. Types of VPC Endpoints

### Gateway Endpoint

Supports:

- Amazon S3
- DynamoDB

---

### Interface Endpoint

Supports:

- Secrets Manager
- SSM
- CloudWatch
- KMS
- SNS
- SQS
- Most AWS services

Uses AWS PrivateLink behind the scenes.

---

# 17. Why Use VPC Endpoints?

Benefits

- Improved security
- Reduced internet exposure
- Lower NAT Gateway data processing costs (for supported services)
- Private AWS network communication

---

# 18. AWS PrivateLink

PrivateLink enables private connectivity to supported AWS services or partner services without exposing traffic to the public internet.

```
Your VPC

↓

PrivateLink

↓

Service Provider
```

---

# 19. VPC Flow Logs

VPC Flow Logs capture network metadata.

They record information such as:

- Source IP
- Destination IP
- Source Port
- Destination Port
- Protocol
- ACCEPT / REJECT

Flow Logs **do not capture packet payloads**.

---

# 20. Where Can Flow Logs Be Sent?

Supported destinations:

- CloudWatch Logs
- Amazon S3

Use them for:

- Security investigations
- Network troubleshooting
- Compliance

---

# 21. Production Scenario 1

Problem

Cannot SSH to EC2.

Investigation

- Public IP
- Internet Gateway
- Route Table
- Security Group
- NACL
- OS Firewall (iptables/firewalld)
- SSH service status

---

# 22. Production Scenario 2

Problem

Application cannot reach RDS.

Check

- Security Groups
- NACLs
- Route Tables
- Database endpoint
- DNS resolution
- Application configuration

---

# 23. Production Scenario 3

Problem

Private EC2 cannot access S3.

Investigation

- NAT Gateway
- Route Table
- Gateway Endpoint
- IAM Role
- Bucket Policy

---

# 24. Production Scenario 4

Problem

VPC Peering created.

Communication still fails.

Check

- Route Tables in both VPCs
- Security Groups
- NACLs
- CIDR overlap
- DNS settings (if private DNS is expected)

---

# 25. Production Scenario 5

Problem

Traffic blocked unexpectedly.

Investigation order:

```
Security Group

↓

NACL

↓

Route Table

↓

Gateway

↓

Application
```

---

# 26. Production Scenario 6

Problem

ALB is healthy but users cannot connect.

Check

- Internet Gateway
- Public Route Table
- ALB Security Group
- Listener
- DNS (Route 53)
- Target Group health

---

# 27. Best Practices

- Keep application servers in private subnets.
- Use Security Groups as the primary firewall.
- Use NACLs only when subnet-level filtering is needed.
- Enable VPC Flow Logs for production VPCs.
- Use VPC Endpoints for private AWS service access.
- Prefer Transit Gateway over a large mesh of VPC Peerings.

---

# 28. Common Mistakes

❌ Opening SSH (22) to `0.0.0.0/0` in production.

❌ Putting databases in public subnets.

❌ Overlapping CIDR ranges between VPCs.

❌ Forgetting to update Route Tables after creating VPC Peering.

❌ Assuming VPC Peering is transitive.

❌ Using NAT Gateway when a VPC Endpoint would provide private connectivity to S3 or DynamoDB.

---

# 29. Interview Questions

## Question 1

What is the difference between a Security Group and a Network ACL?

### Perfect Answer

A Security Group operates at the resource level and is stateful, meaning return traffic is automatically allowed. A Network ACL operates at the subnet level and is stateless, so inbound and outbound rules must both be configured.

---

## Question 2

What is the difference between VPC Peering and Transit Gateway?

### Perfect Answer

VPC Peering creates a direct connection between two VPCs. Transit Gateway provides centralized routing between many VPCs and on-premises networks, making it more scalable for large environments.

---

## Question 3

Why would you use a VPC Endpoint?

### Perfect Answer

A VPC Endpoint allows private connectivity from a VPC to supported AWS services without traversing the public internet or requiring a NAT Gateway for those services.

---

## Question 4

Can Security Groups deny traffic?

### Perfect Answer

No. Security Groups support only allow rules. Explicit deny rules are available only in Network ACLs.

---

## Question 5

What information does a VPC Flow Log contain?

### Perfect Answer

VPC Flow Logs record network metadata such as source and destination IPs, ports, protocol, and whether traffic was accepted or rejected. They do not capture packet contents.

---

# 30. Amazon Follow-up Questions

### Question

Can an EC2 instance have multiple Security Groups?

### Perfect Answer

Yes. Multiple Security Groups can be attached to a resource, and the effective permissions are the union of all allow rules.

---

### Question

Can a Security Group reference another Security Group?

### Perfect Answer

Yes. This is a common practice, for example allowing an RDS Security Group to accept traffic only from an application's Security Group instead of specifying IP addresses.

---

### Question

Why might you choose a Gateway Endpoint over a NAT Gateway for S3?

### Perfect Answer

A Gateway Endpoint provides private connectivity to S3 without sending traffic through the internet. It improves security and can reduce NAT Gateway data processing costs for S3 access.

---

### Question

Can VPC Peering connect overlapping CIDR ranges?

### Perfect Answer

No. VPC Peering requires non-overlapping CIDR blocks.

---

# 31. Hands-on Labs (To Perform Later)

## Lab 1

Create Security Groups for:

- ALB
- Application EC2
- Database

Allow only required communication between them.

---

## Lab 2

Create a custom Network ACL and observe how inbound and outbound rules affect traffic.

---

## Lab 3

Enable VPC Flow Logs and analyze accepted and rejected traffic.

---

## Lab 4

Create two VPCs and connect them using VPC Peering.

Verify communication after updating Route Tables.

---

## Lab 5

Create a Gateway Endpoint for S3.

Verify that a private EC2 instance can access S3 without using a NAT Gateway.

---

## Lab 6

Create an Interface Endpoint for AWS Systems Manager (SSM) or Secrets Manager and verify private connectivity.

---

# 32. One-Page Revision

```
Client
   │
Internet
   │
Internet Gateway
   │
Route Table
   │
NACL
   │
Security Group
   │
EC2
```

Remember:

- Security Group = Resource firewall
- NACL = Subnet firewall
- SG = Stateful
- NACL = Stateless
- VPC Peering = 1:1 private connectivity
- Transit Gateway = Hub-and-spoke networking
- Gateway Endpoint = S3, DynamoDB
- Interface Endpoint = Most AWS services
- Flow Logs = Network metadata

---

# Production Troubleshooting Checklist

When a network issue occurs, investigate in this order:

```
DNS
   │
Route Table
   │
Internet Gateway / NAT Gateway
   │
Security Group
   │
Network ACL
   │
VPC Endpoint
   │
Application
```

Avoid making configuration changes until you've identified which layer is failing.

---

# Think Like a Production Engineer

Don't ask:

> "Which firewall is blocking my traffic?"

Ask:

> "At which network layer is the packet being dropped?"

By tracing the packet through routing, gateways, Security Groups, Network ACLs, and the application, you'll identify the root cause systematically instead of relying on trial and error.

# End of Part 3
