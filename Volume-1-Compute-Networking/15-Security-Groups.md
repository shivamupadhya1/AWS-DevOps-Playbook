# Security Groups

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 15
>
> Security Groups

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Security Groups in depth
- Explain stateful firewall behavior
- Configure inbound and outbound rules
- Use Security Group references
- Design secure production architectures
- Troubleshoot connectivity issues
- Answer advanced interview questions

---

# 1. What is a Security Group?

A Security Group (SG) is a **stateful virtual firewall** attached to an AWS resource.

It controls **who can communicate with the resource** and **which outbound connections the resource can initiate**.

Unlike a traditional firewall, a Security Group **only supports Allow rules**.

---

# 2. Resources That Can Use Security Groups

Security Groups can be attached to many AWS resources, including:

- EC2 Instances
- Application Load Balancers (ALB)
- Network Load Balancers (NLB)
- Amazon RDS
- Amazon ECS Tasks (awsvpc mode)
- Amazon EKS Worker Nodes
- Lambda Functions inside a VPC
- Elastic Network Interfaces (ENIs)

---

# 3. Where Does a Security Group Operate?

A Security Group is attached to the **Elastic Network Interface (ENI)** of a resource.

```
Internet
    │
Internet Gateway
    │
Route Table
    │
Subnet
    │
Network ACL
    │
Elastic Network Interface
    │
Security Group
    │
EC2
```

---

# 4. Stateful Firewall

Security Groups are **stateful**.

This means **return traffic is automatically allowed**.

Example:

```
Laptop

↓

SSH (Port 22)

↓

EC2
```

Once the SSH connection is established,

```
EC2

↓

SSH Response

↓

Laptop
```

No outbound SSH rule is required.

AWS automatically allows the response.

---

# 5. Inbound Rules

Inbound rules define:

> Who is allowed to reach my resource?

Example:

| Protocol | Port | Source |
|----------|------|---------|
| SSH | 22 | 203.0.113.10/32 |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |

---

# 6. Outbound Rules

Outbound rules define:

> Where can my resource initiate connections?

Default:

```
All Traffic

↓

0.0.0.0/0
```

Many organizations restrict outbound rules for security and compliance.

---

# 7. Security Groups Are Allow-Only

Security Groups **cannot explicitly deny traffic**.

If traffic does not match an Allow rule:

```
↓

Implicit Deny
```

There is no "Deny" rule option.

---

# 8. Multiple Security Groups

An EC2 instance can have multiple Security Groups attached.

AWS evaluates all of them together.

Example:

Security Group A

```
Allow SSH
```

Security Group B

```
Allow HTTP
```

Effective permissions:

```
SSH

+

HTTP
```

Rules are combined (logical union).

---

# 9. Security Group References

Instead of allowing traffic from an IP range, you can allow traffic from another Security Group.

Example:

```
ALB Security Group

↓

Application Security Group
```

Rule:

```
Source

ALB-SG

Port

8080
```

Only resources using the ALB Security Group can reach the application.

This is preferred over using public IP addresses.

---

# 10. Production Example

```
Internet
      │
      ▼
ALB (SG-ALB)
      │
      ▼
App EC2 (SG-APP)
      │
      ▼
RDS (SG-DB)
```

Rules:

SG-ALB

```
80
443

↓

Internet
```

SG-APP

```
8080

↓

SG-ALB
```

SG-DB

```
3306

↓

SG-APP
```

The database cannot be accessed directly from the internet.

---

# 11. Rule Evaluation

Traffic arrives.

AWS checks:

```
Security Group

↓

Matching Allow Rule?

↓

YES

↓

Permit

NO

↓

Implicit Deny
```

---

# 12. Security Group Limits

Typical limits (can vary and may be increased via quota requests):

- Rules per Security Group
- Security Groups per ENI
- Security Groups per Region

Always check current AWS service quotas before large-scale designs.

---

# 13. Common Ports

| Service | Port |
|----------|------|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| MySQL | 3306 |
| PostgreSQL | 5432 |
| Oracle | 1521 |
| SQL Server | 1433 |
| Redis | 6379 |
| MongoDB | 27017 |

---

# 14. Production Architecture

```
                Internet
                     │
              ALB (SG-ALB)
                     │
          ┌──────────┴──────────┐
          │                     │
    EC2-1 (SG-APP)        EC2-2 (SG-APP)
          │                     │
          └──────────┬──────────┘
                     │
                 RDS (SG-DB)
```

Only the ALB is internet-facing.

---

# 15. Common Mistakes

❌ Allowing SSH from:

```
0.0.0.0/0
```

---

❌ Opening MySQL to the internet.

---

❌ Allowing all ports from everywhere.

---

❌ Using public IPs instead of Security Group references.

---

❌ Forgetting outbound restrictions in regulated environments.

---

# 16. Best Practices

- Follow the principle of least privilege.
- Use Security Group references whenever possible.
- Separate Security Groups by tier (ALB, App, DB).
- Remove unused rules.
- Review Security Groups regularly.
- Restrict SSH/RDP access to trusted IPs or use AWS Systems Manager Session Manager.

---

# 17. Production Scenarios

## Scenario 1

### Problem

Cannot SSH to EC2.

Check:

- Inbound Port 22
- Source IP
- Public IP
- Route Table
- Internet Gateway
- NACL
- SSH service

---

## Scenario 2

### Problem

ALB Health Checks Fail.

Check:

- Target Group Port
- Application Listening Port
- Application Security Group
- ALB Security Group
- Health Check Path

---

## Scenario 3

### Problem

Application cannot connect to RDS.

Check:

- Database Security Group
- Source Security Group
- Database Port
- Route Table
- NACL

---

## Scenario 4

### Problem

Private EC2 cannot reach the internet.

Security Group allows outbound?

If yes:

Continue checking:

- NAT Gateway
- Route Table
- DNS

---

## Scenario 5

### Problem

Application intermittently loses connectivity.

Check:

- Security Group changes
- Network ACL
- Route Table
- Application logs
- Connection tracking

---

# 18. Security Groups vs Network ACLs

| Feature | Security Group | NACL |
|----------|----------------|------|
| Level | ENI / Resource | Subnet |
| Stateful | Yes | No |
| Deny Rules | No | Yes |
| Default | Implicit Deny | Configurable Allow/Deny |
| Typical Use | Primary firewall | Subnet-level protection |

---

# 19. Interview Questions

## Question 1

What is a Security Group?

### Answer

A Security Group is a stateful virtual firewall attached to an Elastic Network Interface that controls inbound and outbound traffic using allow rules.

---

## Question 2

What does "stateful" mean?

### Answer

When inbound traffic is allowed, the corresponding response traffic is automatically allowed, even if there isn't an explicit outbound rule for that response.

---

## Question 3

Can Security Groups deny traffic?

### Answer

No.

Security Groups support only allow rules. Traffic that doesn't match an allow rule is implicitly denied.

---

## Question 4

Can an EC2 have multiple Security Groups?

### Answer

Yes.

AWS combines all allow rules from the attached Security Groups. If any attached Security Group allows the traffic, it is permitted.

---

## Question 5

Can Security Groups reference other Security Groups?

### Answer

Yes.

This is a common best practice for allowing communication between application tiers without relying on IP addresses.

---

## Question 6

Do Security Groups inspect packet payloads?

### Answer

No.

They evaluate connection metadata such as protocol, port, and source/destination. They are not deep packet inspection firewalls.

---

## Question 7

Does removing an inbound rule terminate existing connections?

### Answer

Existing established connections may continue because Security Groups use connection tracking. New connections matching the removed rule will be blocked.

---

## Question 8

Which is evaluated first: Route Table or Security Group?

### Answer

The Route Table determines where the packet should go. Once the packet reaches the destination resource, the Security Group determines whether the traffic is allowed.

---

# 20. Amazon Follow-up Questions

### Question

Can two EC2 instances in the same Security Group communicate?

### Answer

Yes, if the Security Group contains a rule allowing traffic from itself (self-reference) or another applicable rule.

---

### Question

Can a Security Group span multiple Availability Zones?

### Answer

Yes.

Security Groups are scoped to the VPC, not to a specific Availability Zone.

---

### Question

Can you attach the same Security Group to EC2 and RDS?

### Answer

Yes, provided both resources are in the same VPC and the design makes sense from a security perspective.

---

### Question

Can you edit a Security Group without restarting an EC2?

### Answer

Yes.

Security Group changes take effect dynamically. No reboot is required.

---

# 21. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Create a Security Group allowing:

- SSH from your public IP
- HTTP from anywhere

Verify access.

---

## Lab 2

Create:

- ALB Security Group
- Application Security Group

Allow application access only from the ALB Security Group.

---

## Lab 3

Create an RDS Security Group.

Allow MySQL only from the Application Security Group.

---

## Lab 4

Remove the SSH rule.

Verify that new SSH connections fail.

---

## Lab 5

Attach multiple Security Groups to an EC2 instance and observe how permissions are combined.

---

# 22. One-Page Revision

```
Internet
    │
Route Table
    │
Subnet
    │
Network ACL
    │
Elastic Network Interface
    │
Security Group
    │
EC2
```

Remember:

- Resource-level firewall
- Stateful
- Allow rules only
- Implicit deny
- Multiple SGs = Union of all allow rules
- Security Group references are preferred over IP addresses
- Changes apply immediately

---

# Think Like a Production Engineer

When debugging connectivity, don't assume the Security Group is the problem.

Ask:

1. Is the packet reaching the instance? (Route Table, IGW/NAT, NACL)
2. Does the Security Group allow the traffic?
3. Is the application listening on the expected port?
4. Is the operating system firewall blocking it?

Checking each layer systematically leads to faster and more reliable troubleshooting.

# End of Chapter
