# Transit Gateway

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 18
>
> Transit Gateway – Part 3 (Hybrid Networking)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Connect On-Premises networks to AWS
- Understand Transit Gateway VPN Attachments
- Integrate Direct Connect with TGW
- Understand BGP Route Propagation
- Design Hybrid Cloud Architecture
- Design Highly Available Networks
- Troubleshoot Hybrid Connectivity

---

# 47. Why Hybrid Networking?

Very few enterprise companies run **100% on AWS**.

Most organizations have:

- On-Premise Data Centers
- VMware Infrastructure
- Physical Servers
- Legacy Applications
- Oracle Databases
- SAP Systems

These resources still need to communicate with AWS.

Example:

```
                 Company Network

                +----------------+

                |  Data Center   |

                | 192.168.1.0/24 |

                +-------+--------+

                        │

              VPN / Direct Connect

                        │

                Transit Gateway

                        │

        --------------------------------

        │              │              │

     Production      Dev          Shared

        VPC          VPC          Services
```

Instead of connecting the data center to every VPC separately, connect it once to the Transit Gateway.

---

# 48. Transit Gateway VPN Attachment

A Site-to-Site VPN can terminate directly on the Transit Gateway.

```
Customer Gateway

        │

IPSec VPN

        │

Transit Gateway

        │

All Connected VPCs
```

Advantages:

- One VPN for multiple VPCs
- Centralized routing
- Easier management
- Better scalability

---

# 49. Why Not Create a VPN for Every VPC?

Suppose you have:

```
Production

Development

Testing

Shared Services

Monitoring
```

Without Transit Gateway:

```
VPN → Production

VPN → Development

VPN → Testing

VPN → Shared Services

VPN → Monitoring
```

Five VPNs.

With Transit Gateway:

```
VPN

↓

Transit Gateway

↓

All VPCs
```

Only one VPN attachment is required.

---

# 50. Direct Connect with Transit Gateway

AWS Direct Connect can also terminate on a Transit Gateway.

```
On-Prem

↓

Direct Connect

↓

Direct Connect Gateway

↓

Transit Gateway

↓

All VPCs
```

This is the preferred enterprise architecture.

---

# 51. Why Use Direct Connect?

Compared to VPN:

Benefits include:

- Dedicated private connection
- Lower latency
- Higher bandwidth
- Consistent network performance
- No Internet dependency

---

# 52. Transit Gateway + Direct Connect Architecture

```
             Corporate Data Center

                    │

              Direct Connect

                    │

          Direct Connect Gateway

                    │

            Transit Gateway

        ┌─────────┼─────────┐

        │         │         │

     Prod      Dev      Shared

      VPC       VPC      VPC
```

---

# 53. BGP (Border Gateway Protocol)

BGP is the routing protocol used with:

- AWS Site-to-Site VPN (dynamic routing)
- AWS Direct Connect

Instead of manually configuring routes:

```
10.0.0.0/16

172.16.0.0/16

192.168.0.0/16
```

BGP exchanges them automatically.

---

# 54. Static Routing vs Dynamic Routing

### Static Routing

```
Admin manually creates routes.
```

Advantages:

- Simple
- Predictable

Disadvantages:

- Doesn't scale
- Manual maintenance

---

### Dynamic Routing (BGP)

```
Routers exchange routes automatically.
```

Advantages:

- Automatic updates
- Better failover
- Easier to scale

---

# 55. Route Propagation Example

```
On-Prem

192.168.0.0/16

↓

BGP

↓

Transit Gateway

↓

Production VPC
```

The route is learned automatically through BGP.

---

# 56. High Availability

AWS Site-to-Site VPN provides two VPN tunnels.

```
Customer Router

      │

  Tunnel 1

      │

AWS VPN Endpoint

      │

  Tunnel 2

      │

AWS VPN Endpoint
```

If one tunnel fails, traffic switches to the other.

---

# 57. Enterprise Hybrid Architecture

```
                 Head Office

                       │

              Direct Connect

                       │

         Direct Connect Gateway

                       │

             Transit Gateway

        ┌────────┼────────┬────────┐

        │        │        │        │

     Production  Dev    Shared   Security

        VPC      VPC      VPC      VPC
```

---

# 58. Inspection VPC

Many companies route all traffic through a dedicated Security VPC.

```
Production

      │

Transit Gateway

      │

Inspection VPC

      │

Firewall

      │

Destination
```

Benefits:

- Central firewall
- IDS/IPS
- Packet inspection
- Compliance

---

# 59. Centralized Security

Instead of deploying firewalls inside every VPC:

```
Firewall

↓

Inspection VPC

↓

Transit Gateway

↓

All VPCs
```

Reduces cost and simplifies management.

---

# 60. Disaster Recovery

Example:

```
Mumbai TGW

↓

TGW Peering

↓

Singapore TGW

↓

DR Environment
```

If Mumbai fails:

Traffic can be redirected to Singapore.

---

# 61. Monitoring

Monitor:

- VPN Tunnel Status
- BGP Status
- Transit Gateway Attachments
- Route Tables
- VPC Flow Logs
- CloudWatch Metrics

---

# 62. Best Practices

- Use BGP whenever possible.
- Deploy redundant VPN tunnels.
- Prefer Direct Connect for critical workloads.
- Separate production and development using TGW route tables.
- Monitor attachment health continuously.
- Document network topology.

---

# 63. Common Mistakes

❌ One VPN per VPC.

❌ Ignoring BGP.

❌ No failover testing.

❌ Single Direct Connect connection.

❌ No monitoring.

❌ Mixing production and development routes.

---

# 64. Production Scenario 1

### Problem

On-Premises users cannot reach AWS.

Checklist:

- VPN tunnel status
- Customer Gateway configuration
- Transit Gateway attachment
- Route propagation
- Security Groups
- Network ACLs

---

# 65. Production Scenario 2

### Problem

Direct Connect is established but traffic does not reach the VPC.

Possible causes:

- Missing TGW attachment
- Missing propagation
- Incorrect BGP advertisements
- Incorrect VPC route table

---

# 66. Production Scenario 3

### Problem

VPN Tunnel 1 fails.

Expected behavior:

Tunnel 2 automatically carries traffic if configured correctly.

---

# 67. Interview Questions

## Question 11

Why connect a VPN to Transit Gateway instead of directly to each VPC?

### Answer

A single VPN attachment to the Transit Gateway provides centralized connectivity to multiple VPCs, reducing complexity and simplifying management.

---

## Question 12

Why is BGP preferred?

### Answer

BGP automatically exchanges routes, improves scalability, and supports automatic failover without manual route updates.

---

## Question 13

Can Direct Connect terminate on a Transit Gateway?

### Answer

Yes. Direct Connect integrates with a Direct Connect Gateway, which can connect to a Transit Gateway to provide private connectivity to multiple VPCs.

---

## Question 14

What is an Inspection VPC?

### Answer

An Inspection VPC contains centralized security appliances such as firewalls or IDS/IPS. Traffic from connected VPCs is routed through it for inspection and policy enforcement.

---

## Question 15

How does AWS provide high availability for Site-to-Site VPN?

### Answer

AWS provisions two VPN tunnels. If one tunnel becomes unavailable, traffic can fail over to the second tunnel.

---

# 68. Hands-on Labs

## Lab 11

Create a Transit Gateway.

Attach:

- Production VPC
- Development VPC

---

## Lab 12

Create a Site-to-Site VPN attachment.

Observe propagated routes.

---

## Lab 13

Enable BGP.

Verify automatic route propagation.

---

## Lab 14

Design an Inspection VPC architecture.

---

## Lab 15

Simulate a VPN tunnel failure and observe failover.

---

# 69. One-Page Revision

```
On-Prem

      │

VPN / Direct Connect

      │

Transit Gateway

      │

TGW Route Table

      │

Production / Dev / Shared VPCs
```

Remember:

- One VPN can serve multiple VPCs.
- One Direct Connect can serve multiple VPCs.
- BGP automates routing.
- Use two VPN tunnels for high availability.
- Centralize security with an Inspection VPC.
- Monitor TGW attachments and tunnel health.

---

# Think Like a Production Engineer

Enterprise hybrid networking is built around simplicity and resilience.

Instead of creating separate VPNs or Direct Connect links for every VPC:

1. Centralize connectivity with a Transit Gateway.
2. Use BGP for dynamic routing.
3. Design for high availability with redundant tunnels and connections.
4. Separate environments using TGW route tables.
5. Route sensitive traffic through an Inspection VPC for centralized security.

This architecture is widely used in production because it scales cleanly as the number of VPCs and on-premises networks grows.

# End of Part 3
