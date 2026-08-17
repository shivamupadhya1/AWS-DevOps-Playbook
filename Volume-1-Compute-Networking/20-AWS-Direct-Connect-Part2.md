# AWS Direct Connect

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 20
>
> AWS Direct Connect – Part 1 (Fundamentals)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand AWS Direct Connect
- Explain why enterprises use Direct Connect
- Compare Direct Connect with VPN
- Understand Direct Connect architecture
- Learn Direct Connect components
- Understand Virtual Interfaces (VIFs)
- Explain packet flow

---

# 1. Why Do We Need AWS Direct Connect?

When organizations move workloads to AWS, they often require:

- High bandwidth
- Low latency
- Predictable performance
- Private connectivity
- Regulatory compliance

A Site-to-Site VPN uses the **public Internet**, so performance depends on Internet conditions.

AWS Direct Connect provides a **dedicated private network connection** between your on-premises network and AWS.

---

# 2. What is AWS Direct Connect?

AWS Direct Connect (DX) is a dedicated physical network connection from your data center or office to an AWS Direct Connect location.

Instead of sending traffic over the Internet, traffic flows through a private circuit.

```
Corporate Data Center

        │

Dedicated Circuit

        │

AWS Direct Connect Location

        │

AWS Network

        │

VPC
```

---

# 3. Real Production Example

A bank hosts:

- Core Banking
- Payment Gateway
- Customer Portal

Requirements:

- Low latency
- Stable bandwidth
- Private connectivity

Instead of using only VPN:

```
Bank Data Center

↓

AWS Direct Connect

↓

AWS Cloud
```

---

# 4. Benefits of Direct Connect

- Private connectivity
- Consistent latency
- High bandwidth
- Reduced Internet dependency
- Improved network stability
- Better performance for large data transfers

---

# 5. Direct Connect Architecture

```
Corporate Data Center

        │

Router

        │

Dedicated Fiber

        │

AWS Direct Connect Location

        │

AWS Backbone

        │

AWS Region

        │

VPC
```

---

# 6. AWS Global Backbone

One of the biggest advantages of Direct Connect is that traffic uses the AWS global network after entering AWS.

```
Customer

↓

DX Location

↓

AWS Backbone

↓

Mumbai Region

↓

VPC
```

The public Internet is bypassed for the AWS portion of the journey.

---

# 7. Components of Direct Connect

A Direct Connect deployment includes:

- Customer Router
- Direct Connect Location
- Dedicated Connection
- Virtual Interface (VIF)
- Direct Connect Gateway (optional)
- Virtual Private Gateway (VGW) or Transit Gateway (TGW)

---

# 8. Customer Router

The customer router is located in your:

- Data Center
- Office
- Colocation Facility

It connects to AWS through the Direct Connect circuit.

Examples:

- Cisco
- Juniper
- Arista
- Fortinet

---

# 9. Direct Connect Location

A Direct Connect Location is a physical facility where AWS provides network connectivity.

```
Customer Rack

↓

Cross Connect

↓

AWS Router
```

You connect your equipment to AWS at this location, often through a colocation provider.

---

# 10. Dedicated Connection

The physical link between the customer and AWS is called a **Dedicated Connection**.

Common bandwidth options (availability may vary by location):

- 1 Gbps
- 10 Gbps
- 100 Gbps

Hosted connections with other capacities may also be available through AWS Direct Connect Partners.

---

# 11. Packet Flow

```
User

↓

Corporate LAN

↓

Router

↓

Direct Connect

↓

AWS Router

↓

AWS Backbone

↓

Virtual Interface

↓

VGW / TGW

↓

VPC

↓

EC2
```

---

# 12. Public vs Private Internet

### Internet

```
Corporate

↓

ISP

↓

Internet

↓

AWS
```

Performance varies based on Internet congestion.

---

### Direct Connect

```
Corporate

↓

Dedicated Circuit

↓

AWS

↓

VPC
```

Provides a more consistent private path.

---

# 13. Direct Connect Speeds

Organizations choose speeds based on workload.

Examples:

Small Company

- 1 Gbps

Large Enterprise

- 10 Gbps

Large-scale deployments may use multiple 10 Gbps or 100 Gbps connections depending on requirements.

---

# 14. Direct Connect Availability

AWS recommends:

- Multiple Direct Connect connections
- Different physical locations when possible
- Redundant customer routers

Never rely on a single circuit for critical production workloads.

---

# 15. Direct Connect Use Cases

Typical workloads include:

- Hybrid Cloud
- Backup
- Disaster Recovery
- Database Replication
- Large File Transfers
- SAP
- Oracle Databases
- VMware Cloud
- Financial Applications

---

# 16. Security

Direct Connect provides:

- Private connectivity

However:

Traffic is **not encrypted by default**.

If encryption is required:

- VPN over Direct Connect
- MACsec (where supported)
- Application-level encryption (TLS)

---

# 17. Best Practices

- Deploy redundant connections.
- Use BGP for routing.
- Monitor network utilization.
- Use Transit Gateway for multiple VPCs.
- Document routing.
- Monitor latency and errors.

---

# 18. Common Mistakes

❌ Assuming Direct Connect encrypts traffic.

❌ Deploying only one connection.

❌ Ignoring failover testing.

❌ Forgetting BGP configuration.

❌ No monitoring.

---

# 19. Interview Questions

## Question 1

What is AWS Direct Connect?

### Answer

AWS Direct Connect is a dedicated private network connection between an on-premises network and AWS, providing more consistent performance than Internet-based connectivity.

---

## Question 2

Does Direct Connect use the Internet?

### Answer

The connection between the customer and the Direct Connect location is a dedicated private circuit. After entering AWS, traffic travels over the AWS global network rather than the public Internet.

---

## Question 3

Does Direct Connect encrypt traffic?

### Answer

No.

Direct Connect provides private connectivity but does not encrypt traffic by default. Encryption can be added using VPN over Direct Connect, MACsec (where supported), or application-layer encryption.

---

## Question 4

Why do enterprises use Direct Connect?

### Answer

To obtain consistent network performance, private connectivity, higher bandwidth, and lower latency for critical workloads.

---

## Question 5

What devices are required?

### Answer

Typically:

- Customer Router
- Direct Connect Connection
- Virtual Interface (VIF)
- VGW or TGW (depending on the architecture)

---

# 20. Hands-on Labs

## Lab 1

Draw a Direct Connect architecture connecting an on-premises data center to an AWS VPC.

---

## Lab 2

Compare VPN and Direct Connect for:

- Latency
- Bandwidth
- Cost
- Security
- Availability

---

## Lab 3

Identify where encryption is and is not provided in a Direct Connect architecture.

---

## Lab 4

Design a redundant Direct Connect deployment with two customer routers and two circuits.

---

## Lab 5

Document the packet flow from an on-premises user to an EC2 instance through Direct Connect.

---

# 21. One-Page Revision

```
Corporate Network

↓

Customer Router

↓

Direct Connect Circuit

↓

AWS Direct Connect Location

↓

AWS Backbone

↓

VGW / TGW

↓

VPC
```

Remember:

- Direct Connect = Dedicated private connection.
- Uses AWS global backbone.
- More consistent latency than Internet VPN.
- Does not encrypt traffic by default.
- Use redundant circuits for production.

---

# Think Like a Production Engineer

Direct Connect is the preferred connectivity option for enterprises with predictable, high-volume traffic between on-premises environments and AWS.

When designing a Direct Connect solution:

1. Plan for redundancy from day one.
2. Use BGP for dynamic routing.
3. Encrypt sensitive traffic if required.
4. Monitor circuit health continuously.
5. Integrate with Transit Gateway for scalable hybrid networking.

# End of Part 1
