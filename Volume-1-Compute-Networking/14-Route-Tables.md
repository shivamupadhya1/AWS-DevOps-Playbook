# Route Tables

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 14
>
> Route Tables

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand what a Route Table is
- Understand how AWS routes packets
- Explain Longest Prefix Match
- Configure Public and Private Route Tables
- Understand Local Routes
- Understand Blackhole Routes
- Troubleshoot routing issues
- Design production-ready routing

---

# 1. What is a Route Table?

A Route Table is a set of routing rules (routes) that tells AWS **where to send network traffic**.

Think of it as the **GPS of your VPC**.

Whenever an EC2 sends a packet, AWS checks the Route Table associated with that subnet and decides the next destination.

---

# 2. Real-Life Analogy

Imagine Google Maps.

If you want to travel from Delhi to Mumbai,

Google Maps decides:

```
Delhi

↓

Expressway

↓

Highway

↓

Mumbai
```

A Route Table does the same thing for network packets.

```
Packet

↓

Destination

↓

Next Hop

↓

Target
```

---

# 3. How AWS Uses Route Tables

Suppose an EC2 wants to reach Google.

```
EC2

↓

Route Table

↓

Internet Gateway

↓

Internet
```

If it wants to reach another EC2 in the same VPC:

```
EC2

↓

Route Table

↓

Local Route

↓

Other EC2
```

---

# 4. Components of a Route

Every route has two parts.

| Destination | Target |
|-------------|--------|
| CIDR Block | Next Hop |

Example:

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | igw-12345 |

---

# 5. Local Route

Every VPC automatically gets a Local Route.

Example

```
Destination

10.0.0.0/16

↓

Target

local
```

This allows communication between all subnets inside the VPC.

You cannot delete this route.

---

# 6. Default Route

```
0.0.0.0/0
```

This means:

> "Any destination I don't already know."

Examples:

```
Google

GitHub

Docker Hub

Ubuntu Repository

External APIs
```

Usually points to:

- Internet Gateway
- NAT Gateway
- Virtual Private Gateway
- Transit Gateway

---

# 7. Public Route Table

Example

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

Resources in subnets associated with this Route Table can reach the internet (provided other requirements such as public IPs are met).

---

# 8. Private Route Table

Example

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | NAT Gateway |

Private instances use the NAT Gateway for outbound internet access.

---

# 9. Longest Prefix Match (Very Important)

AWS always selects the **most specific matching route**.

Example Route Table:

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 10.0.1.0/24 | Firewall |
| 0.0.0.0/0 | Internet Gateway |

Packet destination:

```
10.0.1.25
```

Matching routes:

```
10.0.0.0/16

10.0.1.0/24
```

AWS chooses:

```
10.0.1.0/24
```

because `/24` is more specific than `/16`.

---

# 10. Why Longest Prefix Match Matters

Suppose your company wants all traffic to:

```
10.20.0.0/16
```

to pass through a firewall.

You add:

```
10.20.0.0/16

↓

Firewall Appliance
```

Everything else continues using the default route.

This allows selective traffic inspection.

---

# 11. Route Table Association

A Route Table is associated with one or more subnets.

Example:

```
Public Route Table

↓

Public Subnet A

Public Subnet B
```

Another Route Table:

```
Private Route Table

↓

Private Subnet A

Private Subnet B
```

---

# 12. Main Route Table

Every VPC has one Main Route Table.

If a subnet is not explicitly associated with another Route Table, it uses the Main Route Table.

---

# 13. Custom Route Tables

Production environments usually create separate Route Tables for:

- Public Subnets
- Private Application Subnets
- Database Subnets
- Inspection/Firewall Subnets

This keeps routing simple and secure.

---

# 14. Blackhole Route

A route becomes **Blackhole** when its target no longer exists.

Example:

```
0.0.0.0/0

↓

NAT Gateway

↓

NAT Gateway Deleted
```

AWS marks the route as:

```
Blackhole
```

Traffic matching that route is dropped.

---

# 15. Route Targets

Common route targets include:

- Local
- Internet Gateway (IGW)
- NAT Gateway
- Transit Gateway
- VPC Peering Connection
- Virtual Private Gateway (VPN)
- Gateway VPC Endpoint
- Network Interface (rare scenarios)

---

# 16. Packet Flow Example

Public EC2 accessing the internet:

```
EC2

↓

Route Table

↓

0.0.0.0/0

↓

Internet Gateway

↓

Internet
```

Private EC2:

```
EC2

↓

Route Table

↓

0.0.0.0/0

↓

NAT Gateway

↓

Internet Gateway

↓

Internet
```

---

# 17. Production Architecture

```
                    Internet
                         │
                  Internet Gateway
                         │
             Public Route Table
                         │
              Public Subnet (ALB)
                         │
                  NAT Gateway
                         │
             Private Route Table
                         │
         Application Servers / EKS Nodes
                         │
                  Database Subnet
```

---

# 18. Common Mistakes

❌ Forgetting to associate a subnet with the correct Route Table.

❌ Assuming Public IP alone provides internet access.

❌ Deleting the NAT Gateway without updating routes.

❌ Sending database subnet traffic to the Internet Gateway.

❌ Overcomplicating routing with unnecessary custom Route Tables.

---

# 19. Best Practices

- Use separate Route Tables for public and private subnets.
- Keep database subnets isolated.
- Name Route Tables clearly.
- Review routes after infrastructure changes.
- Remove unused routes.
- Monitor for Blackhole routes.

---

# 20. Production Scenarios

## Scenario 1

### Problem

Public EC2 cannot access the internet.

Check:

- Internet Gateway attached?
- Public Route Table?
- Route `0.0.0.0/0` → IGW?
- Public IP?
- Security Group?

---

## Scenario 2

### Problem

Private EC2 cannot install software.

Check:

- Route `0.0.0.0/0` → NAT Gateway?
- NAT Gateway available?
- Internet Gateway attached?
- DNS resolution?

---

## Scenario 3

### Problem

Traffic suddenly stopped after deleting a NAT Gateway.

Root Cause:

The Route Table still points to the deleted NAT Gateway.

AWS marks it as **Blackhole**.

---

## Scenario 4

### Problem

VPC Peering created but communication fails.

Check:

- Route added in VPC A?
- Route added in VPC B?
- Security Groups?
- Non-overlapping CIDRs?

---

## Scenario 5

### Problem

Packets are not following the expected path.

Check:

- Longest Prefix Match
- Route specificity
- Route target
- Route propagation (if using VPN or Transit Gateway)

---

# 21. Troubleshooting Checklist

When network traffic fails:

```
1. Route Table Association

↓

2. Route Entry Exists?

↓

3. Correct Target?

↓

4. Target Healthy?

↓

5. Longest Prefix Match?

↓

6. Security Group

↓

7. Network ACL

↓

8. Application
```

---

# 22. Interview Questions

## Question 1

What is a Route Table?

### Answer

A Route Table is a collection of routing rules that determines where network traffic from a subnet should be directed.

---

## Question 2

What is the Local Route?

### Answer

The Local Route is automatically created by AWS and enables communication between resources within the same VPC.

---

## Question 3

Can one Route Table be associated with multiple subnets?

### Answer

Yes. A single Route Table can be associated with multiple subnets if they require the same routing behavior.

---

## Question 4

What is Longest Prefix Match?

### Answer

When multiple routes match a destination IP, AWS selects the route with the most specific prefix (the longest matching CIDR).

---

## Question 5

What is a Blackhole Route?

### Answer

A Blackhole Route is a route whose target no longer exists, such as a deleted NAT Gateway or VPC Peering Connection. Traffic matching that route is dropped.

---

## Question 6

Can you delete the Local Route?

### Answer

No. AWS automatically manages the Local Route, and it cannot be removed.

---

## Question 7

Can different subnets use different Route Tables?

### Answer

Yes. This is common in production environments to separate public, private, and database traffic.

---

## Question 8

What happens if a subnet has no explicit Route Table association?

### Answer

It automatically uses the VPC's Main Route Table.

---

# 23. Amazon Follow-up Questions

### Question

Can a Route Table have multiple default routes?

### Answer

No. For a given IP family (IPv4 or IPv6), a Route Table typically has one effective default route. AWS prevents conflicting routes with the same destination to different targets.

---

### Question

Does a Route Table filter traffic?

### Answer

No.

It only determines where traffic goes.

Traffic filtering is handled by:

- Security Groups
- Network ACLs

---

### Question

Does the Route Table check port numbers?

### Answer

No.

It only evaluates the destination IP address.

Ports are evaluated later by Security Groups and Network ACLs.

---

### Question

Can a Route Table point to another Route Table?

### Answer

No.

A Route Table points to supported AWS networking targets such as an Internet Gateway, NAT Gateway, Transit Gateway, VPC Peering Connection, or Gateway Endpoint.

---

# 24. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Create:

- Public Route Table
- Private Route Table

Associate them with the appropriate subnets.

---

## Lab 2

Configure:

```
0.0.0.0/0

↓

Internet Gateway
```

Verify internet connectivity from a public EC2.

---

## Lab 3

Configure:

```
0.0.0.0/0

↓

NAT Gateway
```

Verify outbound internet access from a private EC2.

---

## Lab 4

Delete the NAT Gateway and observe the Route Table status.

Look for the **Blackhole** route.

---

## Lab 5

Create a VPC Peering Connection and add the required routes in both VPCs.

Verify connectivity between instances.

---

# 25. One-Page Revision

```
EC2
 │
 ▼
Route Table
 │
 ├── local → Same VPC
 ├── IGW → Internet
 ├── NAT → Outbound Internet
 ├── TGW → Multiple VPCs
 ├── Peering → Another VPC
 └── VPCE → AWS Services
```

Key points:

- Route Tables decide **where** packets go.
- Security Groups decide **whether** traffic is allowed.
- Longest Prefix Match always wins.
- Every VPC has a Main Route Table.
- Every VPC has a Local Route.
- Blackhole routes indicate an invalid or deleted target.

---

# Think Like a Production Engineer

When troubleshooting, don't start with the application.

Start with the packet.

Ask yourself:

1. Which subnet is the instance in?
2. Which Route Table is associated with that subnet?
3. Which route matches the destination?
4. Is the target available?
5. Are Security Groups and NACLs permitting the traffic?

Following the packet step by step is the fastest way to identify routing problems in production.

# End of Chapter
