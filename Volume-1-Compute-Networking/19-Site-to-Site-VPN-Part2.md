# Site-to-Site VPN

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 19
>
> Site-to-Site VPN – Part 2 (Routing, BGP & Tunnel Redundancy)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Configure routing for AWS VPN
- Understand Static vs Dynamic Routing
- Understand Border Gateway Protocol (BGP)
- Configure VPN Tunnel Redundancy
- Understand High Availability
- Troubleshoot Routing Problems
- Explain packet flow in hybrid environments

---

# 26. Routing in Site-to-Site VPN

Creating a VPN tunnel alone is **not enough**.

The routers on both sides must know **where to send traffic**.

Example:

```
On-Premises

192.168.1.0/24

        │

VPN Tunnel

        │

AWS VPC

10.0.0.0/16
```

When an employee accesses:

```
10.0.1.50
```

The on-premises router must know:

```
Destination

10.0.0.0/16

↓

VPN Tunnel
```

Similarly, AWS must know:

```
Destination

192.168.1.0/24

↓

Virtual Private Gateway
```

Without routing, the VPN tunnel may be **UP**, but communication will still fail.

---

# 27. Static Routing

Static routing means routes are manually configured.

Example:

On-Prem Router

```
Destination

10.0.0.0/16

↓

VPN Tunnel
```

AWS Route Table

```
192.168.1.0/24

↓

VGW
```

---

## Advantages

- Easy to understand
- Predictable
- Suitable for small environments

---

## Disadvantages

- Manual updates
- Difficult to maintain
- Poor scalability
- No automatic failover

---

# 28. Dynamic Routing (BGP)

Enterprise environments rarely use static routes.

Instead they use:

```
Border Gateway Protocol (BGP)
```

BGP automatically exchanges routes between AWS and your on-premises router.

---

# 29. What is BGP?

BGP is the routing protocol used between autonomous systems.

Think of it as:

```
Router A

↓

"I know these networks."

↓

Router B

↓

"I know these networks."
```

Instead of manually creating routes, both routers advertise their available networks.

---

# 30. Autonomous System Number (ASN)

Every BGP router belongs to an Autonomous System (AS).

Example:

```
AWS ASN

64512

↓

Corporate ASN

65000
```

These numbers uniquely identify routing domains.

When creating a Customer Gateway, you specify the on-premises ASN if using dynamic routing.

---

# 31. Route Advertisement

Suppose AWS owns:

```
10.0.0.0/16
```

It advertises:

```
10.0.0.0/16
```

to the on-premises router.

Similarly, the on-premises router advertises:

```
192.168.1.0/24
```

to AWS.

This process is automatic with BGP.

---

# 32. BGP Packet Flow

```
Corporate Router

↓

Advertise

192.168.1.0/24

↓

AWS VPN Endpoint

↓

VGW

↓

VPC
```

The reverse process occurs for AWS networks.

---

# 33. VPN Route Propagation

AWS can automatically propagate routes from the VPN into the VPC route table.

Example:

```
Corporate Network

192.168.1.0/24

↓

Route Propagation

↓

VPC Route Table
```

This reduces manual configuration.

---

# 34. Manual Route Example

Without propagation:

```
Destination

192.168.1.0/24

↓

VGW
```

must be added manually.

---

# 35. VPN Tunnel Redundancy

AWS automatically creates:

```
Tunnel 1

Tunnel 2
```

for every Site-to-Site VPN connection.

```
Corporate Router

      │

Tunnel 1

      │

AWS

      │

Tunnel 2

      │

AWS
```

This provides high availability.

---

# 36. Why Two Tunnels?

Imagine:

```
Tunnel 1

↓

Internet Failure
```

Without a second tunnel:

```
VPN Down
```

With two tunnels:

```
Tunnel 1

↓

Fails

↓

Tunnel 2

↓

Traffic Continues
```

---

# 37. Active/Standby vs Active/Active

### Active/Standby

One tunnel carries traffic.

The second tunnel is used only if the first fails.

---

### Active/Active

Both tunnels carry traffic.

This depends on your on-premises device and routing configuration.

---

# 38. VPN Connection States

Possible tunnel states include:

```
UP

DOWN

ESTABLISHING
```

Always verify tunnel health before troubleshooting routing.

---

# 39. Packet Flow Example

```
User

↓

Corporate Router

↓

Tunnel 1

↓

AWS VPN Endpoint

↓

Virtual Private Gateway

↓

VPC Route Table

↓

EC2
```

Every step must be configured correctly.

---

# 40. High Availability Architecture

```
Corporate Router

      │

  Tunnel 1

      │

AWS VPN Endpoint A

      │

Virtual Private Gateway

      │

AWS VPN Endpoint B

      │

Tunnel 2

      │

Corporate Router
```

If one tunnel fails, the second tunnel continues to provide connectivity.

---

# 41. Monitoring VPN

Monitor:

- Tunnel State
- BGP Status
- Route Advertisements
- CloudWatch Metrics
- VPC Flow Logs
- Network Throughput

Useful alarms:

- Tunnel Down
- BGP Session Down
- Packet Drops

---

# 42. Best Practices

- Use BGP instead of static routing where possible.
- Monitor tunnel status continuously.
- Test failover regularly.
- Keep Customer Gateway firmware updated.
- Document routing policies.
- Use non-overlapping CIDR ranges.

---

# 43. Common Mistakes

❌ Configuring only one tunnel.

❌ Blocking UDP 500 or UDP 4500 on the firewall.

❌ Incorrect BGP ASN.

❌ Missing propagated routes.

❌ Incorrect VPC route table.

❌ Overlapping network ranges.

---

# 44. Production Scenario 1

### Problem

Tunnel status is **UP**, but EC2 is unreachable.

Checklist:

- VPC route table
- Security Groups
- Network ACLs
- BGP routes
- Customer Gateway routes

The issue is often routing rather than the VPN tunnel itself.

---

# 45. Production Scenario 2

### Problem

Tunnel 1 goes DOWN.

Expected behavior:

Traffic automatically switches to Tunnel 2 if failover is configured correctly.

---

# 46. Production Scenario 3

### Problem

BGP session never establishes.

Possible causes:

- Incorrect ASN
- Firewall blocking BGP (TCP 179)
- Incorrect tunnel configuration
- Mismatched VPN parameters

---

# 47. Interview Questions

## Question 6

Why does AWS create two VPN tunnels?

### Answer

To provide high availability. If one tunnel becomes unavailable, traffic can continue through the second tunnel, reducing downtime.

---

## Question 7

What is the advantage of BGP over static routing?

### Answer

BGP automatically exchanges routes, supports failover, and scales better than manually maintained static routes.

---

## Question 8

What is an Autonomous System Number (ASN)?

### Answer

An ASN uniquely identifies a routing domain in BGP. AWS and the on-premises router exchange routing information using their ASNs.

---

## Question 9

Can a VPN tunnel be UP while traffic still fails?

### Answer

Yes.

The tunnel may be established, but communication can still fail because of incorrect routing, Security Groups, Network ACLs, or firewall rules.

---

## Question 10

What protocol does BGP use?

### Answer

BGP uses **TCP port 179** to exchange routing information.

---

# 48. Hands-on Labs

## Lab 6

Create a VPN using **dynamic routing (BGP)**.

Observe route propagation.

---

## Lab 7

Configure a VPN using **static routing**.

Compare it with dynamic routing.

---

## Lab 8

Disable one tunnel.

Verify that traffic continues through the second tunnel.

---

## Lab 9

Inspect the propagated routes in the VPC route table.

---

## Lab 10

Use CloudWatch to monitor tunnel health.

---

# 49. One-Page Revision

```
Corporate Network

↓

Customer Gateway

↓

Tunnel 1
Tunnel 2

↓

Virtual Private Gateway

↓

VPC Route Table

↓

EC2
```

Remember:

- Two VPN tunnels are created by AWS.
- BGP automates route exchange.
- Static routing requires manual updates.
- Route propagation reduces administration.
- High availability depends on redundant tunnels and proper routing.

---

# Think Like a Production Engineer

A healthy VPN tunnel does **not** guarantee application connectivity.

When troubleshooting:

1. Verify tunnel status.
2. Check BGP session state.
3. Confirm route propagation.
4. Inspect VPC route tables.
5. Review Security Groups and Network ACLs.
6. Validate the on-premises router configuration.

In enterprise environments, routing issues are more common than tunnel failures. Always verify the entire packet path instead of focusing only on the VPN status.

# End of Part 2
