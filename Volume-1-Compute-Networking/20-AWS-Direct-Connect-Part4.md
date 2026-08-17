# AWS Direct Connect

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 20
>
> AWS Direct Connect – Part 4 (Troubleshooting, Monitoring, Terraform, CLI & Advanced Interview Questions)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Troubleshoot Direct Connect issues
- Monitor Direct Connect connections
- Configure Direct Connect using AWS CLI
- Understand Direct Connect with Terraform
- Diagnose BGP issues
- Design production-ready architectures
- Answer advanced interview questions

---

# 71. Complete Packet Flow

One of the most common interview questions.

Suppose a user in your corporate office opens an application hosted in AWS.

```
User

↓

Corporate LAN

↓

Switch

↓

Firewall

↓

Customer Router

↓

Direct Connect Circuit

↓

AWS Direct Connect Location

↓

AWS Edge Router

↓

Direct Connect Gateway

↓

Transit Gateway

↓

TGW Route Table

↓

Application VPC

↓

Application Load Balancer

↓

ECS / EC2

↓

RDS
```

Every component in this path must be configured correctly.

---

# 72. Direct Connect Troubleshooting Flow

When Direct Connect stops working, troubleshoot in this order:

```
Physical Circuit

↓

Direct Connect Connection

↓

Virtual Interface (VIF)

↓

BGP Session

↓

DX Gateway

↓

Transit Gateway

↓

Route Tables

↓

Security Groups

↓

Network ACLs

↓

Application
```

Never begin by checking the EC2 instance.

Start with the network.

---

# 73. Check Physical Connection

Verify:

- Direct Connect connection is available.
- Customer router interfaces are up.
- Fiber link is healthy.
- Cross-connect is operational.

Possible failures:

- Fiber cut
- Router failure
- Incorrect optical module
- Colocation issue

---

# 74. Verify Virtual Interface (VIF)

Confirm:

- Correct VIF type
  - Private
  - Public
  - Transit
- Correct VLAN
- Correct BGP configuration

Example mistake:

```
Need:

Private VIF

Configured:

Public VIF
```

Result:

Private VPC resources cannot be reached.

---

# 75. Verify BGP

The BGP session should be:

```
Established
```

If not:

Possible causes:

- Incorrect ASN
- Incorrect BGP peer IP
- Firewall blocking TCP 179
- Wrong VLAN
- Incorrect MD5 authentication (if configured)

---

# 76. Verify Route Advertisement

Customer advertises:

```
192.168.10.0/24
```

AWS advertises:

```
10.0.0.0/16
```

If routes are not advertised:

Applications become unreachable even though the Direct Connect circuit is healthy.

---

# 77. Verify Route Tables

Check:

### Transit Gateway Route Table

```
Corporate Network

↓

Production VPC
```

### VPC Route Table

```
Corporate Network

↓

Transit Gateway
```

Missing routes are a common cause of connectivity issues.

---

# 78. Verify Security

Even with routing configured correctly:

Security Groups may block traffic.

Example:

```
HTTPS

443

Blocked
```

Result:

Network connectivity exists.

Application access fails.

---

# 79. Verify Network ACL

Remember:

Network ACLs are **stateless**.

Both:

- Inbound
- Outbound

must allow traffic.

---

# 80. CloudWatch Monitoring

Monitor:

- ConnectionState
- ConnectionBpsIngress
- ConnectionBpsEgress
- ConnectionPpsIngress
- ConnectionPpsEgress
- BGP Status
- Virtual Interface Status

Create alarms for:

- Connection Down
- High bandwidth utilization
- Packet errors
- BGP session failure

---

# 81. AWS CLI Commands

### List Direct Connect Connections

```bash
aws directconnect describe-connections
```

---

### List Virtual Interfaces

```bash
aws directconnect describe-virtual-interfaces
```

---

### List Direct Connect Gateways

```bash
aws directconnect describe-direct-connect-gateways
```

---

### List LAGs (Link Aggregation Groups)

```bash
aws directconnect describe-lags
```

---

### List Hosted Connections

```bash
aws directconnect describe-hosted-connections
```

---

# 82. Terraform Example

## Direct Connect Gateway

```hcl
resource "aws_dx_gateway" "main" {

  name            = "corp-dx-gateway"

  amazon_side_asn = 64512

}
```

---

## Direct Connect Gateway Association

```hcl
resource "aws_dx_gateway_association" "main" {

  dx_gateway_id         = aws_dx_gateway.main.id

  associated_gateway_id = aws_ec2_transit_gateway.main.id

}
```

---

> **Note:** Creating the physical Direct Connect connection itself cannot generally be fully provisioned through Terraform because it requires AWS-side physical infrastructure. Terraform is commonly used to manage logical resources such as DX Gateways and associations after the connection exists.

---

# 83. Link Aggregation Group (LAG)

A **Link Aggregation Group (LAG)** combines multiple Direct Connect connections into one logical connection.

Example:

```
DX 1

DX 2

DX 3

↓

LAG

↓

100 Gbps Logical Connection
```

Benefits:

- Higher bandwidth
- Simplified management
- Improved redundancy

---

# 84. Hosted Connection

Some organizations don't order Direct Connect directly from AWS.

Instead, they purchase a **Hosted Connection** through an AWS Direct Connect Partner.

```
Company

↓

Partner

↓

AWS
```

Benefits:

- Lower initial cost
- Faster provisioning
- Flexible bandwidth options

---

# 85. Production Best Practices

✔ Deploy multiple customer routers.

✔ Deploy multiple Direct Connect circuits.

✔ Use multiple Direct Connect locations.

✔ Configure VPN as backup.

✔ Use Transit Gateway for enterprise environments.

✔ Monitor CloudWatch metrics.

✔ Use Infrastructure as Code where possible.

✔ Document BGP neighbors and route advertisements.

✔ Test failover regularly.

---

# 86. Common Mistakes

❌ Single Direct Connect circuit

❌ One customer router

❌ Incorrect VLAN

❌ Wrong ASN

❌ Wrong VIF type

❌ Missing Transit Gateway routes

❌ No VPN backup

❌ No monitoring

❌ Never testing failover

---

# 87. Production Scenario 1

## Problem

Direct Connect is UP.

Application is unreachable.

### Investigation

Direct Connect ✔

BGP ✔

DX Gateway ✔

Transit Gateway ✔

Security Group ❌

Port 443 blocked.

### Solution

Allow HTTPS traffic.

---

# 88. Production Scenario 2

## Problem

BGP session never establishes.

### Investigation

Customer ASN:

```
65000
```

AWS configured:

```
65100
```

### Root Cause

Incorrect ASN configuration.

---

# 89. Production Scenario 3

## Problem

Users cannot access Amazon S3.

### Investigation

Private VIF configured.

### Root Cause

S3 access requires a **Public VIF** (or an S3 Gateway VPC Endpoint for traffic originating from within a VPC).

---

# 90. Production Scenario 4

## Problem

One Direct Connect circuit fails.

Expected Behavior:

Traffic automatically shifts to:

- Second Direct Connect circuit, or
- Site-to-Site VPN

depending on BGP routing preferences and redundancy design.

---

# 91. Production Scenario 5

## Problem

One Direct Connect location experiences an outage.

Expected Behavior

Traffic moves to the second Direct Connect location if redundant connectivity has been configured.

---

# 92. Decision Matrix

| Requirement | Recommended Solution |
|------------|----------------------|
| One VPC | Private VIF + VGW |
| Multiple VPCs | Transit VIF + TGW |
| AWS Public Services | Public VIF |
| Enterprise | DX Gateway + TGW |
| Backup | Site-to-Site VPN |
| High Availability | Dual DX + Dual Router |

---

# 93. Advanced Interview Questions

## Question 19

What is the difference between Direct Connect and VPN?

### Answer

Direct Connect uses a dedicated private circuit, while VPN uses the public Internet with IPSec encryption.

---

## Question 20

Does Direct Connect provide encryption?

### Answer

No.

Traffic is private but not encrypted by default.

Encryption can be added using:

- VPN over Direct Connect
- MACsec (supported scenarios)
- TLS/Application encryption

---

## Question 21

What is a Virtual Interface?

### Answer

A Virtual Interface (VIF) is a logical interface on a Direct Connect connection used to route traffic to AWS.

---

## Question 22

What is the difference between Private, Public and Transit VIF?

### Answer

Private VIF:

Connects to private VPC resources.

Public VIF:

Connects to AWS public service endpoints.

Transit VIF:

Connects Direct Connect to a Transit Gateway through a Direct Connect Gateway.

---

## Question 23

What is a Direct Connect Gateway?

### Answer

A Direct Connect Gateway provides centralized connectivity between Direct Connect and one or more Virtual Private Gateways or Transit Gateways.

---

## Question 24

Why is BGP used?

### Answer

BGP dynamically exchanges routes and supports scalable routing and automatic failover.

---

## Question 25

Can Direct Connect fail?

### Answer

Yes.

Hardware, fiber, router, or facility failures can occur.

Therefore, AWS recommends redundant connections and VPN backup.

---

## Question 26

What is Link Aggregation Group (LAG)?

### Answer

LAG combines multiple Direct Connect connections into one logical connection to increase bandwidth and resilience.

---

## Question 27

How do you troubleshoot Direct Connect?

### Answer

Check:

- Physical connection
- VIF
- BGP
- Route advertisement
- DX Gateway
- Transit Gateway
- Route tables
- Security Groups
- Network ACLs
- Application

---

## Question 28

Why do enterprises use Transit Gateway with Direct Connect?

### Answer

Transit Gateway provides centralized routing and allows one Direct Connect connection to serve multiple VPCs and AWS accounts.

---

# 94. Hands-on Labs

## Lab 16

Design:

- Two customer routers
- Two Direct Connect circuits
- Transit Gateway
- VPN backup

Explain failover.

---

## Lab 17

Create a packet flow diagram from:

Corporate User

↓

Application running in ECS

through Direct Connect.

---

## Lab 18

Compare:

- Private VIF
- Public VIF
- Transit VIF

Create a comparison table.

---

## Lab 19

Simulate a BGP failure.

Document troubleshooting steps.

---

## Lab 20

Create a complete hybrid cloud architecture using:

- Direct Connect
- VPN
- Transit Gateway
- Shared Services VPC
- Production VPC
- Development VPC
- Inspection VPC

---

# 95. One-Page Revision

```
Corporate Network

↓

Customer Router

↓

Direct Connect

↓

Virtual Interface

↓

DX Gateway

↓

Transit Gateway

↓

Application VPC

↓

ALB

↓

ECS

↓

RDS

↓

VPN Backup
```

Remember:

- Direct Connect = Dedicated private connection.
- Uses BGP for dynamic routing.
- Three VIF types:
  - Private
  - Public
  - Transit
- DX Gateway centralizes connectivity.
- Transit Gateway scales to multiple VPCs.
- Always deploy redundant circuits.
- Keep VPN as a backup.
- Monitor CloudWatch metrics.

---

# Chapter Summary

In this chapter, you learned:

- Direct Connect fundamentals
- Components and architecture
- Private, Public, and Transit VIFs
- Direct Connect Gateway
- Transit Gateway integration
- BGP routing
- High availability
- VPN backup
- Monitoring
- Troubleshooting
- AWS CLI commands
- Terraform examples
- Production best practices
- Enterprise architectures
- Advanced interview questions

You should now be able to confidently design, implement, troubleshoot, and explain AWS Direct Connect in both real-world production environments and technical interviews.

---

# Think Like a Production Engineer

Direct Connect is not just about getting a private connection to AWS—it is about building a resilient hybrid network.

When designing production infrastructure:

1. Never rely on a single circuit.
2. Always use BGP for dynamic routing.
3. Use Transit Gateway for scalability.
4. Configure VPN as a backup path.
5. Monitor connection health continuously.
6. Test failover regularly.
7. Automate logical networking resources with Infrastructure as Code.

Production-grade networking is measured by how well it handles failures—not just how well it works under normal conditions.

# End of Chapter 20 – AWS Direct Connect
