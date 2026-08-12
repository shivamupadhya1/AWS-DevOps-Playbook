# VPC Peering

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 17
>
> VPC Peering

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand VPC Peering
- Explain why VPC Peering is needed
- Design VPC-to-VPC communication
- Configure Route Tables
- Understand Cross-Region Peering
- Understand Cross-Account Peering
- Explain limitations
- Troubleshoot VPC Peering
- Answer DevOps Interview Questions

---

# 1. Why Do We Need VPC Peering?

Imagine a company has two AWS VPCs.

```
VPC-A

CIDR

10.0.0.0/16
```

Application Servers

and

```
VPC-B

CIDR

172.16.0.0/16
```

Database Servers

Now,

The application needs to connect to the database.

Without connectivity,

Communication fails.

AWS provides

```
VPC Peering
```

to connect two VPCs privately.

---

# 2. What is VPC Peering?

VPC Peering is a networking connection between two VPCs that enables private communication using private IP addresses.

Traffic never traverses the public internet.

---

# 3. Real Production Example

```
Production VPC

10.0.0.0/16

↓

Peering

↓

Shared Services VPC

172.16.0.0/16
```

Production applications use services like:

- Active Directory
- DNS
- Monitoring
- Logging
- Artifact repositories

hosted in the Shared Services VPC.

---

# 4. VPC Peering Architecture

```
                +----------------------+
                |      AWS Region      |
                +----------------------+

        +----------------+      +----------------+
        |     VPC-A      |      |     VPC-B      |
        |10.0.0.0/16     |      |172.16.0.0/16   |
        |                |      |                |
        |  EC2           |      |  RDS           |
        +--------+-------+      +-------+--------+
                 \                  /
                  \                /
                   \              /
                 VPC Peering Connection
```

Both VPCs remain independent.

Only network connectivity is established.

---

# 5. What Does VPC Peering Provide?

After peering:

- Private IP communication
- Low latency
- High bandwidth
- AWS backbone network
- No Internet Gateway required
- No NAT Gateway required

---

# 6. Before VPC Peering

```
EC2

10.0.1.10

↓

???

↓

Database

172.16.1.20
```

AWS has no route.

Packets are dropped.

---

# 7. After VPC Peering

```
EC2

↓

Private Route

↓

Peering

↓

Database
```

Traffic flows entirely over AWS's private network.

---

# 8. Types of VPC Peering

AWS supports:

### Same Region Peering

```
Mumbai

↓

Mumbai
```

Most common.

---

### Cross Region Peering

```
Mumbai

↓

Singapore
```

Useful for:

- Disaster Recovery
- Global Applications
- Multi-region architectures

---

# 9. Requirements for VPC Peering

Both VPCs must have:

- Non-overlapping CIDR blocks
- Route table updates
- Security Group rules
- NACL rules (if applicable)

If CIDRs overlap:

```
10.0.0.0/16

↓

10.0.0.0/16
```

Peering cannot be established.

---

# 10. Why Overlapping CIDRs Are Not Allowed?

Suppose:

```
VPC-A

10.0.1.5
```

and

```
VPC-B

10.0.1.5
```

When a packet is destined for `10.0.1.5`, AWS cannot determine which VPC should receive it.

Therefore,

Overlapping CIDRs are not supported.

---

# 11. Components Required

A working VPC Peering setup requires:

- Two VPCs
- Peering Connection
- Route Tables
- Security Groups
- Network ACLs (if restricting traffic)

Missing any of these can prevent connectivity.

---

# 12. Route Tables

Creating a peering connection does **not** automatically create routes.

You must update the route table.

Example

VPC-A Route Table

| Destination | Target |
|-------------|--------|
|172.16.0.0/16|Peering Connection|

---

VPC-B Route Table

| Destination | Target |
|-------------|--------|
|10.0.0.0/16|Peering Connection|

Without these routes:

Traffic will fail.

---

# 13. Security Groups

Security Groups must also allow the required traffic.

Example

Application:

```
10.0.0.0/16
```

Database:

```
172.16.0.0/16
```

Security Group

Allow:

```
3306

Source

10.0.0.0/16
```

Now MySQL traffic can flow.

---

# 14. Network ACL

If your subnet uses restrictive NACLs,

remember to allow:

- Inbound traffic
- Outbound traffic
- Ephemeral ports

Otherwise packets may be dropped before reaching the instance.

---

# 15. DNS Resolution

By default,

EC2 instances communicate using private IP addresses.

You can optionally enable private DNS resolution across the peering connection for supported use cases, allowing instances to resolve private addresses instead of public ones.

---

# 16. Typical Use Cases

- Shared Services VPC
- Centralized Monitoring
- Centralized Logging
- Application to Database communication
- Shared DNS
- Internal APIs

---

# 17. Advantages

- Private communication
- High performance
- Simple setup
- No VPN required
- No Internet exposure
- Uses AWS backbone network

---

# 18. Limitations

- No overlapping CIDRs
- Manual route updates
- No automatic transitive routing
- Limited scalability compared to Transit Gateway

We'll discuss these in detail in Part 2.

---

# 19. Best Practices

- Plan CIDR blocks before deployment.
- Use least privilege in Security Groups.
- Keep route tables simple.
- Document all peering relationships.
- Monitor connectivity.

---

# 20. Common Mistakes

❌ Forgetting route table entries.

❌ Forgetting Security Group rules.

❌ Overlapping CIDRs.

❌ Assuming peering is transitive.

❌ Forgetting NACL rules.

---

# 21. Interview Questions

## Question 1

What is VPC Peering?

### Answer

VPC Peering is a private networking connection between two VPCs that allows resources in both VPCs to communicate using private IP addresses over the AWS backbone network.

---

## Question 2

Does VPC Peering use the Internet?

### Answer

No.

Traffic stays on the AWS private backbone network and does not traverse the public internet.

---

## Question 3

Can two VPCs with overlapping CIDRs be peered?

### Answer

No.

AWS requires non-overlapping CIDR ranges because overlapping address spaces create routing ambiguity.

---

## Question 4

Does creating a peering connection automatically update route tables?

### Answer

No.

You must manually add routes in both VPCs pointing to the peering connection.

---

## Question 5

Is an Internet Gateway required for VPC Peering?

### Answer

No.

VPC Peering provides private connectivity without requiring an Internet Gateway or NAT Gateway.

---

# 22. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Create:

- VPC-A (`10.0.0.0/16`)
- VPC-B (`172.16.0.0/16`)

---

## Lab 2

Launch one EC2 instance in each VPC.

---

## Lab 3

Create a VPC Peering Connection.

---

## Lab 4

Update route tables in both VPCs.

---

## Lab 5

Modify Security Groups to allow ICMP and SSH.

Verify connectivity using private IP addresses.

---

# 23. One-Page Revision

```
VPC-A
10.0.0.0/16
     │
     ▼
VPC Peering
     ▲
     │
VPC-B
172.16.0.0/16
```

Remember:

- Private connectivity
- Non-overlapping CIDRs
- Manual route table updates
- Security Groups required
- NACLs may need updates
- No Internet Gateway required
- No NAT Gateway required
- Uses AWS backbone network

---

# Think Like a Production Engineer

When VPC Peering doesn't work, don't immediately blame AWS.

Follow this order:

1. Verify the peering connection is **Active**.
2. Check both route tables.
3. Verify Security Group rules.
4. Check Network ACLs.
5. Ensure CIDR ranges do not overlap.
6. Test connectivity using private IPs.

Most production issues are caused by missing routes or firewall rules—not the peering connection itself.

# End of Part 1
