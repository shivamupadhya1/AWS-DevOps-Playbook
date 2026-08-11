# Internet Gateway (IGW)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 12
>
> Internet Gateway

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand what an Internet Gateway is
- Understand how it works internally
- Understand packet flow
- Differentiate Internet Gateway vs NAT Gateway
- Design internet-facing architectures
- Troubleshoot connectivity issues
- Answer production-level interview questions

---

# 1. What is an Internet Gateway?

An Internet Gateway (IGW) is a highly available, horizontally scaled AWS-managed component that allows communication between a VPC and the public internet.

It performs two primary functions:

- Enables inbound internet traffic to resources with public IPs.
- Enables outbound internet traffic from resources in public subnets.

Without an Internet Gateway, resources inside a VPC **cannot communicate directly with the internet**, even if they have a public IP address.

---

# 2. Why Do We Need an Internet Gateway?

Consider an EC2 instance hosting a web application.

Users on the internet need to access:

```
https://example.com
```

Without an Internet Gateway:

```
User

↓

Internet

↓

❌ No Path

↓

EC2
```

The traffic has no route into your VPC.

---

# 3. Internet Gateway Architecture

```
                    Internet
                        │
                        │
                Internet Gateway
                        │
                Route Table (0.0.0.0/0)
                        │
                 Public Subnet
                        │
                  Security Group
                        │
                      EC2
```

---

# 4. Where is the Internet Gateway Attached?

Important interview point:

The Internet Gateway is attached to the **VPC**, not to a subnet or an EC2 instance.

```
VPC

├── Public Subnet
├── Private Subnet
└── Internet Gateway
```

Only one Internet Gateway can be attached to a VPC at a time.

---

# 5. Does an Internet Gateway Give Internet Access Automatically?

No.

All of the following must be true:

- Internet Gateway attached to the VPC.
- Route Table has `0.0.0.0/0 → IGW`.
- Instance has a Public IP or Elastic IP.
- Security Group allows the traffic.
- Network ACL allows the traffic.

Missing any one of these can break connectivity.

---

# 6. Packet Flow (Inbound)

Example:

A user browses:

```
https://myapp.com
```

Packet flow:

```
Browser

↓

Internet

↓

Internet Gateway

↓

Route Table

↓

Subnet

↓

Security Group

↓

EC2
```

---

# 7. Packet Flow (Outbound)

Example:

The EC2 downloads updates.

```
EC2

↓

Security Group

↓

Route Table

↓

Internet Gateway

↓

Internet
```

---

# 8. Public Subnet and Internet Gateway

A subnet is public only if:

- It uses a Route Table with a default route (`0.0.0.0/0`) pointing to the Internet Gateway.
- Instances have public addressing when internet access is required.

A public subnet without an IGW route behaves like a private subnet.

---

# 9. Public IP Requirement

Even with an Internet Gateway, an EC2 instance **without a Public IP (or Elastic IP)** cannot be reached directly from the internet.

Example:

```
Private IP

10.0.1.20

↓

No Public IP

↓

Internet cannot reach it
```

---

# 10. Elastic IP

Production systems often require a fixed public IP.

Elastic IPs provide:

- Static IPv4 address
- Reusable across supported resources
- Stable endpoint for external systems

Common use cases:

- Bastion Hosts
- NAT Gateways
- Legacy integrations requiring IP allowlists

---

# 11. Internet Gateway vs NAT Gateway

| Feature | Internet Gateway | NAT Gateway |
|----------|------------------|-------------|
| Internet Access | Yes | Outbound only |
| Inbound Connections | Yes | No |
| Used In | Public Subnets | Public Subnet (serving Private Subnets) |
| Requires Public IP on EC2 | Yes | No (for private instances) |
| Managed by AWS | Yes | Yes |

---

# 12. High Availability

An Internet Gateway is a managed AWS service.

You do not:

- Scale it
- Patch it
- Create multiple instances for HA

AWS automatically provides high availability within the Region.

---

# 13. Production Architecture

```
                     Internet
                          │
                    Internet Gateway
                          │
                   Application Load Balancer
                          │
          ┌───────────────┴───────────────┐
          │                               │
     Public Subnet A                 Public Subnet B
          │                               │
      NAT Gateway A                   NAT Gateway B
          │                               │
     Private Subnet A               Private Subnet B
          │                               │
      Application EC2               Application EC2
```

The Internet Gateway serves as the entry point for internet traffic.

---

# 14. Common Misconceptions

❌ "Attaching an Internet Gateway makes all instances public."

False.

Instances also need:

- Correct Route Table
- Public IP or Elastic IP
- Security Group rules
- Network ACL rules

---

❌ "A Public IP alone is enough."

False.

Without an attached Internet Gateway and proper routing, internet traffic cannot enter or leave the VPC.

---

# 15. Production Scenarios

## Scenario 1

### Problem

Cannot SSH to EC2.

### Investigation

- Internet Gateway attached?
- Route Table points to IGW?
- Public IP present?
- Security Group allows TCP/22?
- NACL allows traffic?
- SSH service running?

---

## Scenario 2

### Problem

Website returns timeout.

### Investigation

- Internet Gateway
- Route Table
- ALB
- Target Group
- Security Group
- EC2 health
- Application logs

---

## Scenario 3

### Problem

EC2 cannot reach the internet.

### Investigation

- Public IP assigned?
- Route Table configured?
- Internet Gateway attached?
- Security Group outbound rules?
- NACL outbound rules?

---

## Scenario 4

### Problem

Browser shows "Connection Refused."

### Investigation

- Web server running?
- Security Group allows 80/443?
- Correct listener?
- Application bound to correct interface?
- OS firewall (iptables/firewalld)?

---

# 16. Best Practices

- Keep only internet-facing resources in public subnets.
- Use an ALB instead of exposing application servers directly.
- Use Elastic IPs only when a fixed public IP is required.
- Minimize the number of directly reachable EC2 instances.
- Place databases in private subnets.

---

# 17. Common Mistakes

❌ Forgetting to attach the Internet Gateway.

❌ Forgetting the `0.0.0.0/0` route.

❌ Launching EC2 without a Public IP.

❌ Exposing backend application servers directly to the internet.

❌ Opening unnecessary ports to `0.0.0.0/0`.

---

# 18. Troubleshooting Flow

When an EC2 cannot communicate with the internet:

```
Step 1

Public IP?

↓

Step 2

Internet Gateway attached?

↓

Step 3

Route Table correct?

↓

Step 4

Security Group

↓

Step 5

Network ACL

↓

Step 6

Application

↓

Step 7

Operating System Firewall
```

This sequence helps isolate issues from the network layer down to the application.

---

# 19. Interview Questions

## Question 1

What is an Internet Gateway?

### Answer

An Internet Gateway is an AWS-managed, highly available component attached to a VPC that enables communication between the VPC and the public internet.

---

## Question 2

Can a VPC have multiple Internet Gateways?

### Answer

No. A VPC can have only one attached Internet Gateway at a time.

---

## Question 3

Can an EC2 access the internet with only a Public IP?

### Answer

No. It also needs an attached Internet Gateway, a Route Table directing internet traffic to the IGW, and appropriate Security Group and NACL rules.

---

## Question 4

Can a private subnet use an Internet Gateway directly?

### Answer

Typically no. Private subnets route outbound internet traffic through a NAT Gateway instead of directly to the Internet Gateway.

---

## Question 5

Does an Internet Gateway perform NAT?

### Answer

It enables communication between public IPs and the internet. For private instances requiring outbound internet access, AWS recommends using a NAT Gateway rather than relying on the Internet Gateway.

---

# 20. Amazon Follow-up Questions

### Question

Can an Internet Gateway be shared between VPCs?

### Answer

No. An Internet Gateway is attached to a single VPC.

---

### Question

Can you detach an Internet Gateway from a running VPC?

### Answer

Yes. However, internet connectivity for that VPC will stop until another Internet Gateway is attached.

---

### Question

Does an Internet Gateway have bandwidth limits?

### Answer

AWS manages the Internet Gateway as a highly scalable service. Customers do not configure or manually scale its bandwidth.

---

# 21. Hands-on Labs (When Your AWS Account is Ready)

### Lab 1

Create a VPC and attach an Internet Gateway.

---

### Lab 2

Create a public subnet and associate a Route Table with `0.0.0.0/0 → IGW`.

---

### Lab 3

Launch an EC2 instance with a Public IP.

Verify:

- SSH access
- Ping (where applicable)
- HTTP/HTTPS connectivity

---

### Lab 4

Detach the Internet Gateway.

Observe how internet connectivity changes.

---

### Lab 5

Remove the default route from the Route Table and test connectivity.

---

# 22. One-Page Revision

```
Internet
    │
Internet Gateway
    │
Route Table
    │
Public Subnet
    │
Security Group
    │
EC2
```

Requirements for internet access:

- Internet Gateway attached
- Public IP or Elastic IP
- Route Table → `0.0.0.0/0`
- Security Group allows traffic
- Network ACL allows traffic

---

# Think Like a Production Engineer

Don't stop after confirming that an EC2 has a Public IP.

Trace the entire request path:

```
Client
   │
Internet
   │
Internet Gateway
   │
Route Table
   │
Subnet
   │
Security Group
   │
Operating System Firewall
   │
Application
```

Every production connectivity issue can be narrowed down by validating each layer in this order.

# End of Chapter
