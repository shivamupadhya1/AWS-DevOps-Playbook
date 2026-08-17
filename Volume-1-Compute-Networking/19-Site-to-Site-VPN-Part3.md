# Site-to-Site VPN

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 19
>
> Site-to-Site VPN – Part 3 (Transit Gateway, Hybrid Cloud & Direct Connect Comparison)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Connect Site-to-Site VPN with Transit Gateway
- Design Hybrid Cloud Architecture
- Compare VGW vs TGW
- Compare VPN vs Direct Connect
- Design enterprise production networks
- Understand Disaster Recovery networking
- Explain real-world hybrid architectures

---

# 50. Virtual Private Gateway vs Transit Gateway

One of the most frequently asked interview questions.

## Virtual Private Gateway (VGW)

```
                Corporate Office

                      │

                  VPN Tunnel

                      │

                    VGW

                      │

                  One AWS VPC
```

Characteristics:

- Supports one VPC
- Simple architecture
- Small environments
- Less scalable

---

## Transit Gateway (TGW)

```
             Corporate Office

                    │

                VPN Tunnel

                    │

             Transit Gateway

        ┌────────┼────────┬────────┐

        │        │        │        │

     Prod      Dev     Shared    Security

      VPC       VPC      VPC       VPC
```

Characteristics:

- Supports multiple VPCs
- Enterprise architecture
- Centralized routing
- Highly scalable

---

# 51. When Should You Use VGW?

Choose VGW when:

- Only one VPC needs connectivity
- Small organization
- Proof of Concept
- Development environment
- Small production deployments

Example:

```
Company

↓

VPN

↓

AWS VPC
```

Simple and cost-effective.

---

# 52. When Should You Use Transit Gateway?

Choose Transit Gateway when:

- Multiple VPCs
- Multiple AWS Accounts
- Shared Services
- Centralized Networking
- Hybrid Cloud
- Enterprise Architecture

Example:

```
Production

Development

Testing

Monitoring

Security

↓

Transit Gateway

↓

Corporate Network
```

---

# 53. VPN using Transit Gateway

Instead of:

```
VPN

↓

VPC-A

VPN

↓

VPC-B

VPN

↓

VPC-C
```

Use:

```
Corporate Office

       │

VPN Attachment

       │

Transit Gateway

 ┌─────┼──────┐

 │     │      │

Prod  Dev   Shared
```

Advantages:

- One VPN
- One routing point
- Easy management
- Better scalability

---

# 54. Enterprise Hybrid Cloud Architecture

```
                Corporate Data Center

                     Active Directory

                     Oracle Database

                     SAP Servers

                            │

                  Site-to-Site VPN

                            │

                    Transit Gateway

     ------------------------------------------------

     │               │               │

 Production      Development     Shared Services

     │               │               │

 ECS / EKS        Lambda       Monitoring
```

This architecture is common in banking, healthcare, and large enterprises.

---

# 55. Shared Services Architecture

A Shared Services VPC contains services used by all other VPCs.

Example:

```
Shared Services

↓

DNS

↓

Active Directory

↓

Nexus

↓

GitLab

↓

Monitoring

↓

Logging
```

Every application VPC connects through the Transit Gateway.

---

# 56. Disaster Recovery Architecture

Primary Region

```
Mumbai

↓

Transit Gateway
```

↓

VPN

↓

Corporate Office

↓

VPN

↓

Transit Gateway

↓

Singapore Region
```

Benefits:

- Business Continuity
- Disaster Recovery
- Regional Failover

---

# 57. VPN vs Direct Connect

| Feature | Site-to-Site VPN | Direct Connect |
|----------|-----------------|----------------|
| Connection | Internet | Dedicated Circuit |
| Encryption | Yes | No (Encryption can be added separately, e.g., MACsec on supported links or VPN over DX) |
| Cost | Lower | Higher |
| Latency | Variable | More consistent |
| Bandwidth | Internet dependent | Dedicated bandwidth |
| Deployment Time | Hours | Days to weeks |
| Availability | High (with redundant tunnels) | High (with redundant circuits) |

---

# 58. Which One Should You Choose?

### VPN

Use when:

- Small company
- Temporary connectivity
- Development
- Disaster Recovery
- Quick deployment

---

### Direct Connect

Use when:

- Financial institutions
- Healthcare
- Large enterprises
- High bandwidth
- Low latency
- Mission-critical systems

---

# 59. VPN + Direct Connect

Many enterprises use both.

```
Corporate Office

        │

Direct Connect

        │

Primary

        │

Transit Gateway

        │

VPN

        │

Backup
```

If Direct Connect fails:

Traffic automatically switches to VPN (depending on routing preferences and failover configuration).

---

# 60. Packet Flow (Enterprise)

```
Employee

↓

Corporate LAN

↓

Firewall

↓

Customer Gateway

↓

VPN Tunnel

↓

Transit Gateway

↓

TGW Route Table

↓

Production VPC

↓

Application Load Balancer

↓

ECS

↓

RDS
```

---

# 61. Security Considerations

Always:

✔ Use strong IPSec encryption

✔ Rotate Pre-Shared Keys

✔ Enable CloudTrail

✔ Monitor VPN health

✔ Restrict Security Groups

✔ Enable VPC Flow Logs

✔ Use least privilege IAM

---

# 62. High Availability

Recommended architecture

```
Corporate Router A

       │

Tunnel 1

       │

AWS VPN Endpoint A

=========================

AWS

=========================

AWS VPN Endpoint B

       │

Tunnel 2

       │

Corporate Router B
```

This removes single points of failure on both the AWS and on-premises sides.

---

# 63. Best Practices

- Prefer BGP over static routing.
- Use Transit Gateway for multiple VPCs.
- Keep CIDR blocks unique.
- Monitor VPN continuously.
- Test failover regularly.
- Document routing.
- Enable logging.

---

# 64. Common Mistakes

❌ Using VGW for enterprise-scale environments.

❌ One VPN per VPC.

❌ No backup tunnel testing.

❌ Ignoring BGP.

❌ Overlapping CIDRs.

❌ No monitoring.

---

# 65. Production Scenario 1

### Problem

A company has:

- Production
- Development
- Testing
- Shared Services

Each VPC has its own VPN.

Problem:

Network administration becomes difficult.

Solution:

Replace individual VPNs with:

```
Transit Gateway

+

One VPN Attachment
```

---

# 66. Production Scenario 2

### Problem

Direct Connect fails.

Expected behavior:

Traffic should fail over to the VPN connection if routing priorities and redundancy are configured correctly.

---

# 67. Production Scenario 3

### Problem

Disaster Recovery region becomes active.

Expected behavior:

Corporate users continue accessing applications through the alternate VPN or Direct Connect path based on the disaster recovery design.

---

# 68. Production Scenario 4

### Problem

Corporate users can reach the Application Load Balancer but not the database.

Checklist:

- Security Groups
- Route Tables
- Network ACLs
- Transit Gateway Route Tables
- VPN Routing
- Database Security Group

---

# 69. Interview Questions

## Question 11

When would you use a Transit Gateway instead of a Virtual Private Gateway?

### Answer

Use Transit Gateway when connecting multiple VPCs, multiple AWS accounts, or hybrid environments requiring centralized routing.

---

## Question 12

Can a VPN connect directly to a Transit Gateway?

### Answer

Yes.

AWS supports Site-to-Site VPN attachments directly to a Transit Gateway.

---

## Question 13

Why do enterprises use both Direct Connect and VPN?

### Answer

Direct Connect provides the primary dedicated connection, while VPN provides encrypted backup connectivity for high availability.

---

## Question 14

Is Direct Connect encrypted?

### Answer

Direct Connect is a private dedicated connection but does not encrypt traffic by default. Encryption can be added separately (for example, VPN over Direct Connect or MACsec on supported connections).

---

## Question 15

Can one VPN connect multiple VPCs?

### Answer

Yes.

When the VPN terminates on a Transit Gateway, a single VPN attachment can provide connectivity to multiple attached VPCs.

---

# 70. Hands-on Labs

## Lab 11

Create:

- Transit Gateway
- Production VPC
- Development VPC

Attach both VPCs.

---

## Lab 12

Create a Site-to-Site VPN attachment.

Verify route propagation.

---

## Lab 13

Design a hybrid architecture with:

- Corporate Network
- Transit Gateway
- Three VPCs

Draw the routing diagram.

---

## Lab 14

Compare:

VGW vs TGW

Write five situations where each should be used.

---

## Lab 15

Design a highly available network using:

- Direct Connect
- VPN
- Transit Gateway

Explain the failover process.

---

# 71. One-Page Revision

```
Small Company

VPN

↓

VGW

↓

One VPC

----------------------------

Enterprise

VPN

↓

Transit Gateway

↓

Multiple VPCs

↓

Shared Services
```

Remember:

- VGW → Single VPC
- TGW → Multiple VPCs
- VPN → Encrypted Internet connection
- Direct Connect → Dedicated private connection
- VPN + DX → Common enterprise design
- Transit Gateway centralizes hybrid networking

---

# Think Like a Production Engineer

As your AWS environment grows, avoid scaling by adding more individual VPNs.

Instead:

1. Centralize connectivity through a Transit Gateway.
2. Use BGP for dynamic routing.
3. Keep Direct Connect as the primary path for critical workloads.
4. Configure VPN as a resilient backup.
5. Test failover regularly instead of assuming it works.
6. Document routing and network topology so operational teams can troubleshoot quickly.

A well-designed hybrid network is simple, scalable, resilient, and easy to operate.

# End of Part 3
