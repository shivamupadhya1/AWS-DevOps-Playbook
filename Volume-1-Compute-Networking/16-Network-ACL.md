# Network ACL (NACL)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 16
>
> Network Access Control Lists (NACL)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Network ACLs
- Explain Stateless Firewall
- Understand Rule Evaluation
- Explain Rule Numbers
- Understand Ephemeral Ports
- Compare NACL vs Security Groups
- Troubleshoot Production Networking Problems
- Answer Senior DevOps Interview Questions

---

# 1. What is a Network ACL?

A Network ACL (Network Access Control List) is a **stateless firewall** that operates at the **Subnet level**.

Unlike Security Groups, a NACL protects **every resource inside the subnet**.

Think of it as:

```
Security Guard at Building Entrance
```

Everyone entering the building is checked.

---

# 2. Where Does NACL Work?

```
Internet
      │
Internet Gateway
      │
Route Table
      │
Network ACL
      │
Subnet
      │
Security Group
      │
EC2
```

Notice:

NACL is evaluated **before** traffic reaches the EC2.

---

# 3. Why Does AWS Have Both Security Groups and NACLs?

Security Group

↓

Protects Individual Resources

NACL

↓

Protects Entire Subnet

Example

```
Subnet

├── EC2-1

├── EC2-2

├── ALB

└── ECS Tasks
```

One NACL protects everything inside.

---

# 4. Stateless Firewall

The most important interview topic.

Suppose:

Laptop

↓

SSH

↓

EC2

NACL allows

```
Inbound Port 22
```

The response packet:

```
EC2

↓

Laptop
```

NACL does **NOT** remember the original request.

Therefore,

You must also allow the **outbound response**.

This is why NACLs are called **Stateless**.

---

# 5. Stateful vs Stateless

Security Group

```
Request

↓

Allowed

↓

Response

↓

Automatically Allowed
```

Network ACL

```
Request

↓

Allowed

↓

Response

↓

Must Match Outbound Rule
```

---

# 6. Rule Numbers

Every NACL rule has a Rule Number.

Example

| Rule | Type | Action |
|------|------|--------|
|100|SSH|Allow|
|110|HTTP|Allow|
|120|HTTPS|Allow|
|32767|All|Deny|

AWS evaluates from **lowest number to highest**.

First matching rule wins.

---

# 7. Why Rule Numbers Matter

Suppose:

Rule 100

```
Allow HTTP
```

Rule 200

```
Deny Everything
```

HTTP is allowed because Rule 100 matches first.

Now reverse them.

Rule 100

```
Deny Everything
```

Rule 200

```
Allow HTTP
```

HTTP is denied.

The Allow rule is never reached.

---

# 8. Default NACL

Every VPC gets a Default NACL.

Default behavior

Inbound

```
Allow All
```

Outbound

```
Allow All
```

---

# 9. Custom NACL

A newly created Custom NACL starts with:

Inbound

```
Deny All
```

Outbound

```
Deny All
```

You must explicitly add Allow rules.

---

# 10. Inbound Rules

Example

| Rule | Port | Source | Action |
|------|------|---------|--------|
|100|22|Office IP|Allow|
|110|80|0.0.0.0/0|Allow|
|120|443|0.0.0.0/0|Allow|
|32767|All|All|Deny|

---

# 11. Outbound Rules

Example

| Rule | Port | Destination | Action |
|------|------|-------------|--------|
|100|1024-65535|0.0.0.0/0|Allow|
|110|80|0.0.0.0/0|Allow|
|120|443|0.0.0.0/0|Allow|
|32767|All|All|Deny|

---

# 12. What are Ephemeral Ports?

This is a favourite interview question.

When your laptop connects to:

```
EC2

Port 22
```

Your laptop chooses a temporary source port.

Example

```
Laptop

Source Port

54021

↓

Destination Port

22
```

Response

```
EC2

Source Port

22

↓

Destination Port

54021
```

That temporary port is called an **Ephemeral Port**.

Operating systems typically use dynamic port ranges (commonly starting around **1024**; exact ranges vary by OS).

---

# 13. Why Ephemeral Ports Matter

Suppose:

Inbound

```
22 Allowed
```

Outbound

```
22 Allowed
```

SSH still fails.

Why?

Because the return traffic is going to:

```
54021
```

NOT

```
22
```

Therefore,

Outbound must allow the ephemeral port range.

---

# 14. Typical Linux Example

```
Laptop

54021

↓

EC2

22
```

Response

```
22

↓

54021
```

Without allowing ephemeral destination ports in the appropriate direction, the response packet is dropped.

---

# 15. Common NACL Design

Public Subnet

```
Allow

80

443

22

Ephemeral Ports
```

Private Subnet

```
Allow

Application Port

Database Port

Ephemeral Ports
```

---

# 16. Production Architecture

```
Internet
      │
Internet Gateway
      │
Public NACL
      │
Public Subnet
      │
ALB
      │
Private NACL
      │
Application
      │
Database
```

---

# 17. NACL vs Security Group

| Feature | Security Group | NACL |
|----------|---------------|------|
|Scope|Resource|Subnet|
|Stateful|Yes|No|
|Allow Rules|Yes|Yes|
|Deny Rules|No|Yes|
|Rule Order|Not Applicable|Lowest Rule Number Wins|
|Connection Tracking|Yes|No|

---

# 18. When Should You Use NACL?

Most AWS environments rely primarily on Security Groups.

Use NACLs when:

- Compliance requires subnet-level filtering.
- You want to block a malicious IP range before it reaches any instance.
- You need explicit DENY rules.
- You want an additional layer of defense.

---

# 19. Best Practices

- Keep NACLs simple.
- Use Security Groups as the primary firewall.
- Document rule numbers.
- Leave gaps between rule numbers (100, 110, 120...) for future changes.
- Test after modifying NACL rules.

---

# 20. Common Mistakes

❌ Forgetting outbound ephemeral ports.

❌ Creating overlapping rules with confusing priorities.

❌ Relying only on NACLs instead of Security Groups.

❌ Blocking internal subnet communication accidentally.

❌ Using NACLs for application-level security.

---

# 21. Production Scenarios

## Scenario 1

### Problem

SSH Timeout

Check:

- Security Group
- NACL Inbound
- NACL Outbound
- Ephemeral Ports
- Route Table

---

## Scenario 2

### Problem

Website Loads Slowly

Check:

- NACL Drops
- ALB
- Route Table
- Security Group
- Application Logs

---

## Scenario 3

### Problem

Application Cannot Reach Database

Check:

- NACL between App and DB subnets
- Security Groups
- Database Port
- Route Tables

---

## Scenario 4

### Problem

Health Checks Fail

Possible causes

- ALB Security Group
- Target Security Group
- NACL
- Application Port

---

## Scenario 5

### Problem

Traffic Randomly Fails

Investigation

- Rule Numbers
- Ephemeral Ports
- NACL Associations
- Route Tables

---

# 22. Troubleshooting Flow

```
Route Table

↓

Network ACL

↓

Security Group

↓

Operating System Firewall

↓

Application
```

---

# 23. Interview Questions

## Question 1

What is a Network ACL?

### Answer

A Network ACL is a stateless firewall that controls inbound and outbound traffic at the subnet level using ordered allow and deny rules.

---

## Question 2

Why is NACL Stateless?

### Answer

A Network ACL does not maintain connection state. Every packet is evaluated independently, so response traffic must be explicitly allowed.

---

## Question 3

What happens if both Rule 100 and Rule 200 match?

### Answer

AWS evaluates rules in ascending order. The first matching rule is applied, and later rules are ignored.

---

## Question 4

Can NACL Deny Traffic?

### Answer

Yes.

Unlike Security Groups, NACLs support explicit DENY rules.

---

## Question 5

Why are Ephemeral Ports important?

### Answer

Client devices use temporary source ports when initiating connections. Return traffic is sent to those ports, so NACL rules must allow the appropriate ephemeral port range.

---

## Question 6

Security Group vs NACL?

### Answer

Security Groups are stateful, resource-level firewalls with allow rules only. NACLs are stateless, subnet-level firewalls supporting both allow and deny rules with ordered evaluation.

---

## Question 7

When would you use a NACL?

### Answer

For subnet-level filtering, explicit deny rules, compliance requirements, or to block known malicious IP ranges before traffic reaches resources.

---

## Question 8

Can multiple subnets share the same NACL?

### Answer

Yes.

A single NACL can be associated with multiple subnets, but each subnet can have only one NACL associated at a time.

---

# 24. Amazon Follow-up Questions

### Question

Can one subnet have multiple NACLs?

### Answer

No.

A subnet can be associated with only one Network ACL at a time.

---

### Question

Can a NACL protect RDS?

### Answer

Yes.

If the RDS instance resides in the subnet associated with that NACL, the NACL applies to its network traffic.

---

### Question

Can NACL inspect packet contents?

### Answer

No.

NACLs inspect network metadata (IP addresses, ports, protocol), not application payloads.

---

### Question

Does changing a NACL require an EC2 reboot?

### Answer

No.

Changes take effect immediately for traffic entering or leaving the associated subnet.

---

# 25. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Create a custom NACL.

Associate it with a public subnet.

---

## Lab 2

Allow:

- SSH
- HTTP
- HTTPS
- Ephemeral Ports

Verify connectivity.

---

## Lab 3

Remove the ephemeral port rule.

Observe how return traffic fails.

---

## Lab 4

Add a DENY rule for your own public IP.

Verify that access is blocked.

---

## Lab 5

Create separate NACLs for:

- Public Subnet
- Application Subnet
- Database Subnet

Test communication between tiers.

---

# 26. One-Page Revision

```
Internet
     │
Internet Gateway
     │
Route Table
     │
Network ACL
     │
Subnet
     │
Security Group
     │
EC2
```

Remember:

- Subnet-level firewall
- Stateless
- Supports Allow and Deny
- Rule order matters
- First matching rule wins
- Explicitly allow ephemeral ports
- One NACL per subnet
- A NACL can be shared across multiple subnets

---

# Think Like a Production Engineer

When traffic fails, don't immediately assume the application is broken.

Trace the packet:

```
Client
   │
Route Table
   │
Network ACL
   │
Security Group
   │
Operating System Firewall
   │
Application
```

If the packet is dropped before it reaches the instance, investigate routing and NACLs. If it reaches the instance but fails, investigate Security Groups, the OS firewall, and the application.

# End of Chapter
