# Amazon Virtual Private Cloud (VPC)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 09
>
> Part 2 – Routing, Internet Connectivity & Traffic Flow

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand Route Tables
- Understand Internet Gateway (IGW)
- Understand NAT Gateway
- Understand Elastic IP
- Understand Public IP vs Private IP
- Explain packet flow
- Design internet-facing and private architectures
- Troubleshoot VPC connectivity issues

---

# 1. How Does Networking Work in AWS?

Every packet entering or leaving an EC2 instance follows a path.

```
Client

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

Understanding this flow is essential for debugging.

---

# 2. Route Table

A Route Table is a collection of routing rules.

It tells AWS:

> "If traffic is destined for this network, send it here."

Think of it as the GPS for your VPC.

---

# 3. Default Route

Every VPC has a Local Route.

Example

```
Destination

10.0.0.0/16

Target

local
```

This allows communication between all subnets inside the VPC.

Without this route, instances inside the VPC couldn't communicate.

---

# 4. Internet Gateway (IGW)

An Internet Gateway connects a VPC to the public internet.

It enables:

- Inbound internet traffic
- Outbound internet traffic

Without an IGW:

- No internet access
- No SSH from your laptop
- No public website

---

# 5. Internet Gateway Architecture

```
Internet

↓

Internet Gateway

↓

Route Table

↓

Public Subnet

↓

EC2
```

The IGW is attached to the VPC, not to individual subnets.

---

# 6. Public Route

Example Route Table

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

`0.0.0.0/0` means "all IPv4 destinations."

Traffic not destined for the local VPC is sent to the IGW.

---

# 7. What Makes a Subnet Public?

A subnet is public when:

- Its Route Table has a route to the Internet Gateway.
- The EC2 instance has a Public IP (or Elastic IP).
- Security Groups and NACLs allow the traffic.

All three conditions are required.

---

# 8. Public IP

A Public IP allows an EC2 instance to communicate directly with the internet.

Example

```
Private IP

10.0.1.15

↓

Public IP

13.x.x.x
```

AWS maps the Public IP to the Private IP.

---

# 9. Private IP

Private IPs are used only within private networks.

Example ranges

```
10.x.x.x

172.16.x.x

192.168.x.x
```

Private IPs are not routable over the internet.

---

# 10. Elastic IP (EIP)

An Elastic IP is a static public IPv4 address.

Unlike an auto-assigned Public IP, an Elastic IP remains the same until you release it.

Use cases:

- Bastion Host
- NAT Gateway
- Stable public endpoints
- Legacy systems requiring fixed IPs

---

# 11. Public IP vs Elastic IP

| Public IP | Elastic IP |
|-----------|------------|
| Dynamic | Static |
| Changes on stop/start (unless configured otherwise) | Remains the same |
| Automatically assigned | Manually allocated |
| Good for temporary workloads | Good for production endpoints |

---

# 12. Outbound Internet Flow (Public EC2)

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

# 13. Inbound Internet Flow

```
Internet

↓

Internet Gateway

↓

Route Table

↓

Security Group

↓

EC2
```

---

# 14. NAT Gateway

Private instances often need outbound internet access to:

- Download software
- Install OS updates
- Pull Docker images
- Access AWS APIs

They should **not** be directly reachable from the internet.

A NAT Gateway solves this problem.

---

# 15. NAT Gateway Architecture

```
Internet

↓

Internet Gateway

↓

NAT Gateway (Public Subnet)

↓

Private Route Table

↓

Private EC2
```

---

# 16. Why NAT Gateway?

Without NAT Gateway

```
Private EC2

↓

No Internet Access
```

With NAT Gateway

```
Private EC2

↓

NAT Gateway

↓

Internet
```

Outbound works.

Inbound remains blocked.

---

# 17. Why Put NAT Gateway in a Public Subnet?

A NAT Gateway itself requires internet connectivity.

Therefore:

- NAT Gateway → Public Subnet
- Application Servers → Private Subnet

---

# 18. Route Table for Private Subnet

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | NAT Gateway |

Traffic destined for the internet is sent to the NAT Gateway instead of directly to the IGW.

---

# 19. Public vs Private Traffic Flow

### Public EC2

```
Internet

↓

IGW

↓

Public EC2
```

### Private EC2

```
Internet

↓

IGW

↓

NAT Gateway

↓

Private EC2
```

---

# 20. Typical Production Architecture

```
                    Internet
                        │
                 Internet Gateway
                        │
                Public Subnet (AZ-A)
               ┌───────────────────┐
               │ ALB               │
               │ NAT Gateway       │
               └───────────────────┘
                        │
                Private Subnet (AZ-A)
               ┌───────────────────┐
               │ Application EC2   │
               └───────────────────┘

                Public Subnet (AZ-B)
               ┌───────────────────┐
               │ ALB               │
               │ NAT Gateway       │
               └───────────────────┘
                        │
                Private Subnet (AZ-B)
               ┌───────────────────┐
               │ Application EC2   │
               └───────────────────┘
```

---

# 21. Best Practices

- Keep application servers in private subnets.
- Expose only the Load Balancer to the internet.
- Deploy NAT Gateways in each Availability Zone for high availability.
- Use Elastic IPs only when a fixed public IP is required.
- Avoid assigning Public IPs to backend services.

---

# 22. Common Mistakes

❌ Creating a public subnet without attaching an Internet Gateway.

❌ Expecting a private subnet to access the internet without a NAT Gateway.

❌ Putting databases in public subnets.

❌ Using one NAT Gateway for multiple AZs without considering availability and cross-AZ data transfer costs.

❌ Assuming a Public IP alone provides internet access without proper routing.

---

# 23. Production Scenarios

## Scenario 1

Problem

Cannot SSH to EC2.

Check:

- Public IP or Elastic IP
- Route Table
- Internet Gateway
- Security Group
- NACL

---

## Scenario 2

Problem

Private EC2 cannot install packages.

Check:

- NAT Gateway
- Route Table
- Security Group
- Internet Gateway
- DNS resolution

---

## Scenario 3

Problem

Website inaccessible.

Check:

- ALB
- Listener
- Target Group
- Route Table
- Internet Gateway
- Security Group

---

## Scenario 4

Problem

Application cannot access the internet.

Check:

- Subnet type
- Route Table
- NAT Gateway
- Security Group
- NACL

---

# 24. Interview Questions

## Question 1

What makes a subnet public?

### Perfect Answer

A subnet is public when its Route Table points `0.0.0.0/0` to an Internet Gateway, the instance has a Public or Elastic IP, and the security rules permit the traffic.

---

## Question 2

Why can't a private EC2 access the internet?

### Perfect Answer

A private subnet has no direct route to an Internet Gateway. Outbound internet access requires a NAT Gateway (or NAT Instance) and an appropriate Route Table.

---

## Question 3

Why is a NAT Gateway placed in a public subnet?

### Perfect Answer

Because the NAT Gateway must itself communicate with the internet through an Internet Gateway in order to forward outbound requests from private instances.

---

## Question 4

What is the difference between a Public IP and an Elastic IP?

### Perfect Answer

A Public IP is usually dynamic and may change, while an Elastic IP is a static public IPv4 address that remains associated until it is explicitly released.

---

## Question 5

Can a private EC2 receive inbound traffic from the internet through a NAT Gateway?

### Perfect Answer

No. A NAT Gateway supports outbound connections initiated by private instances. It does not allow unsolicited inbound internet traffic.

---

# 25. Amazon Follow-up Questions

### Question

Can an EC2 with a Public IP but no Internet Gateway access the internet?

### Perfect Answer

No. A Public IP alone is not enough. The VPC must have an attached Internet Gateway, and the Route Table must direct internet-bound traffic to it.

---

### Question

Can two subnets share the same Route Table?

### Perfect Answer

Yes. Multiple subnets can be associated with the same Route Table if they require identical routing behavior.

---

### Question

What happens if you delete the Internet Gateway?

### Perfect Answer

Internet connectivity is lost for resources that depend on it. Internal VPC communication continues because the local route remains intact.

---

# 26. Hands-on Labs (To Perform Later)

## Lab 1

Create a VPC with one public and one private subnet.

---

## Lab 2

Attach an Internet Gateway and verify internet access from the public subnet.

---

## Lab 3

Create a NAT Gateway and configure the private Route Table.

---

## Lab 4

Launch EC2 instances in both subnets and compare connectivity.

---

## Lab 5

Stop the NAT Gateway and observe how outbound internet access from the private subnet fails.

---

# 27. One-Page Revision

```
Internet
    │
Internet Gateway
    │
Public Route Table
    │
Public Subnet
    │
ALB / Bastion / NAT Gateway
    │
Private Route Table
    │
Private Subnet
    │
Application
```

Remember:

- Route Table = Traffic direction
- Internet Gateway = Internet connectivity
- NAT Gateway = Outbound internet for private subnets
- Public Subnet = Route to IGW
- Private Subnet = Route to NAT Gateway
- Elastic IP = Static public IP

---

# Think Like a Production Engineer

Don't ask:

> "Does this instance have a Public IP?"

Ask:

> "Can I trace the complete network path from the client to the instance and back?"

That approach helps you identify routing, gateway, or security issues systematically instead of guessing.

# End of Part 2
