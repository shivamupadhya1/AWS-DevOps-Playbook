# AWS Direct Connect

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 20
>
> AWS Direct Connect – Part 3 (High Availability, Hybrid Architectures & Production Design)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Design highly available Direct Connect architectures
- Understand Direct Connect resiliency models
- Compare Active-Active vs Active-Passive
- Design enterprise hybrid cloud connectivity
- Understand Direct Connect Failover
- Integrate VPN with Direct Connect
- Design Disaster Recovery architectures

---

# 48. High Availability in Direct Connect

A common misconception is:

> "I have Direct Connect, so my network is highly available."

This is **not true**.

A single Direct Connect connection is a **Single Point of Failure (SPOF).**

Example:

```
Corporate Router

      │

Direct Connect

      │

AWS
```

If:

- Fiber is cut
- Router fails
- DX device fails
- Colocation facility has issues

Entire connectivity is lost.

---

# 49. AWS Recommended Architecture

AWS recommends:

```
           Corporate Network

                  │

        ---------------------

        │                   │

   Customer Router A   Customer Router B

        │                   │

      DX 1              DX 2

        │                   │

AWS DX Location A     AWS DX Location B

        │                   │

      AWS Backbone Network

                │

        Direct Connect Gateway

                │

        Transit Gateway

                │

        Production VPC

        Development VPC

        Shared Services
```

Advantages

- No Single Point of Failure
- High Availability
- Better Disaster Recovery
- Load Distribution

---

# 50. Types of Redundancy

AWS recommends redundancy at multiple layers.

### Device Redundancy

Two customer routers.

```
Router A

Router B
```

---

### Circuit Redundancy

Two Direct Connect circuits.

```
DX 1

DX 2
```

---

### Location Redundancy

Two Direct Connect locations.

```
Mumbai DX

Delhi DX
```

or

```
Mumbai DX

Chennai DX
```

---

### Region Redundancy

For Disaster Recovery:

```
Mumbai Region

↓

Singapore Region
```

---

# 51. Active-Active Architecture

Both Direct Connect circuits carry traffic.

```
          Router A

              │

             DX1

              │

AWS

              │

             DX2

              │

          Router B
```

Advantages

- Better bandwidth utilization
- Load sharing
- Faster failover

Disadvantages

- More complex routing
- Requires careful BGP configuration

---

# 52. Active-Passive Architecture

Only one circuit carries traffic.

```
Primary DX

↓

Traffic

Backup DX

↓

Idle
```

If the primary fails:

```
Backup becomes active.
```

Advantages

- Simpler
- Easier troubleshooting

Disadvantages

- Backup bandwidth remains unused during normal operation

---

# 53. VPN as Backup

A common enterprise design:

```
Corporate Network

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

Traffic automatically switches to VPN using BGP routing preferences.

---

# 54. Direct Connect + VPN Packet Flow

Normal Operation

```
Corporate

↓

Direct Connect

↓

AWS
```

Failure

```
Corporate

↓

VPN Tunnel

↓

AWS
```

Users continue working with minimal interruption.

---

# 55. Disaster Recovery Architecture

```
Corporate Office

         │

     Direct Connect

         │

 Mumbai Region

         │

 Transit Gateway

         │

 Production

==============================

Disaster

↓

Singapore Region

↓

Transit Gateway

↓

Backup Environment
```

If Mumbai becomes unavailable:

Applications are served from Singapore.

---

# 56. Multi-Account Architecture

Most enterprises use AWS Organizations.

```
Networking Account

↓

Transit Gateway

↓

Production Account

↓

Development Account

↓

Security Account

↓

Shared Services
```

Direct Connect connects only once to the Networking Account.

All other accounts access it through the Transit Gateway.

---

# 57. Shared Services Architecture

Shared Services VPC

```
DNS

Active Directory

Nexus

GitLab

Monitoring

Logging

Jump Server
```

Application VPCs access these services through the Transit Gateway.

---

# 58. Inspection VPC

Many enterprises inspect all traffic.

```
Corporate

↓

Direct Connect

↓

Transit Gateway

↓

Inspection VPC

↓

Firewall

↓

Application VPC
```

Benefits

- Central Firewall
- IDS
- IPS
- Compliance
- Logging

---

# 59. Hybrid Cloud Architecture

```
Head Office

↓

Direct Connect

↓

Transit Gateway

↓

Production

↓

EKS

↓

ECS

↓

Lambda

↓

RDS

↓

Shared Services
```

On-premises applications communicate seamlessly with cloud workloads.

---

# 60. Large Enterprise Example

Imagine a bank.

```
Branches

↓

Head Office

↓

Direct Connect

↓

Transit Gateway

↓

Payment Systems

↓

Banking APIs

↓

Fraud Detection

↓

Monitoring

↓

Security
```

Thousands of employees access AWS privately without relying on the public Internet.

---

# 61. Monitoring

Monitor:

- Direct Connect Connection State
- BGP Status
- Virtual Interfaces
- CloudWatch Metrics
- Network Throughput
- Errors
- Packet Drops

Create alarms for:

- Connection Down
- BGP Down
- High Error Rate
- High Utilization

---

# 62. Best Practices

✔ Always deploy redundant Direct Connect circuits.

✔ Use Transit Gateway for multiple VPCs.

✔ Use BGP for routing.

✔ Test VPN failover regularly.

✔ Document every advertised route.

✔ Monitor CloudWatch metrics.

✔ Keep CIDRs non-overlapping.

✔ Design for future expansion.

---

# 63. Common Mistakes

❌ One Direct Connect circuit.

❌ One customer router.

❌ No VPN backup.

❌ No BGP monitoring.

❌ No Disaster Recovery design.

❌ Forgetting to test failover.

---

# 64. Production Scenario 1

## Problem

Direct Connect suddenly fails.

Expected Behavior

Traffic automatically switches to VPN.

No application outage.

---

# 65. Production Scenario 2

## Problem

One Direct Connect location loses connectivity.

Expected Behavior

Traffic automatically shifts to the second Direct Connect location if redundancy is configured.

---

# 66. Production Scenario 3

## Problem

Corporate users cannot access Production VPC.

Investigation

DX Connection ✔

BGP ✔

DX Gateway ✔

Transit Gateway ✔

TGW Route Table ❌ Missing propagation

Root Cause

Transit Gateway routing was incomplete.

---

# 67. Production Scenario 4

## Problem

Disaster Recovery site activated.

Expected Behavior

Traffic reaches Singapore Region using the preplanned disaster recovery routing.

---

# 68. Interview Questions

## Question 13

Why should Direct Connect not be deployed with only one connection?

### Answer

A single Direct Connect circuit creates a Single Point of Failure. AWS recommends redundant circuits and customer routers for production environments.

---

## Question 14

What is Active-Active Direct Connect?

### Answer

Both Direct Connect circuits carry traffic simultaneously, providing load sharing and faster failover.

---

## Question 15

What is Active-Passive Direct Connect?

### Answer

Only the primary circuit carries traffic. The secondary circuit remains idle until the primary fails.

---

## Question 16

Can Direct Connect use VPN as backup?

### Answer

Yes.

Many organizations use Direct Connect as the primary path and Site-to-Site VPN as the backup path.

---

## Question 17

Why do enterprises use Transit Gateway with Direct Connect?

### Answer

Transit Gateway centralizes connectivity to multiple VPCs and AWS accounts, reducing complexity and improving scalability.

---

## Question 18

How do you achieve high availability with Direct Connect?

### Answer

- Multiple customer routers
- Multiple Direct Connect circuits
- Multiple Direct Connect locations
- BGP
- VPN backup
- Transit Gateway

---

# 69. Hands-on Labs

## Lab 11

Design a highly available Direct Connect architecture with:

- Two routers
- Two DX circuits
- Transit Gateway
- VPN backup

---

## Lab 12

Draw an Active-Active Direct Connect architecture.

---

## Lab 13

Draw an Active-Passive Direct Connect architecture.

---

## Lab 14

Design a Disaster Recovery network using:

- Mumbai
- Singapore
- Direct Connect
- Transit Gateway

---

## Lab 15

Create a complete hybrid architecture including:

- Corporate Office
- Direct Connect
- VPN Backup
- Transit Gateway
- Shared Services
- Three VPCs

---

# 70. One-Page Revision

```
Corporate Network

↓

Router A

↓

DX 1

↓

AWS

↓

DX 2

↓

Router B

↓

DX Gateway

↓

Transit Gateway

↓

Production

↓

Development

↓

Shared Services

↓

VPN Backup
```

Remember:

- Never rely on one Direct Connect.
- Use redundant routers.
- Use redundant circuits.
- Use multiple DX locations.
- Configure VPN backup.
- Use BGP for automatic failover.
- Transit Gateway simplifies enterprise networking.

---

# Think Like a Production Engineer

Enterprise connectivity is designed for **failure**, not just success.

When building a Direct Connect solution:

1. Assume a fiber cut can happen at any time.
2. Assume a router can fail.
3. Assume a Direct Connect location may become unavailable.
4. Provide redundant paths with BGP.
5. Keep a Site-to-Site VPN ready as a backup.
6. Test failover regularly instead of assuming it works.

A production-grade Direct Connect architecture is one where users continue working even when individual components fail.

# End of Part 3
