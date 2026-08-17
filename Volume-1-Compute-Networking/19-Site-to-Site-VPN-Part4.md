# Site-to-Site VPN

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 19
>
> Site-to-Site VPN – Part 4 (Troubleshooting, Monitoring, Terraform, AWS CLI & Production Guide)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Troubleshoot Site-to-Site VPN issues
- Monitor VPN tunnels
- Configure VPN using AWS CLI
- Configure VPN using Terraform
- Design production-ready VPN architectures
- Answer advanced interview questions
- Understand AWS best practices

---

# 72. Complete Packet Flow

One of the most important interview questions.

Suppose an employee accesses an application hosted in AWS.

```
User PC

↓

Corporate LAN

↓

Switch

↓

Firewall

↓

Customer Gateway

↓

Tunnel 1 / Tunnel 2

↓

AWS VPN Endpoint

↓

Virtual Private Gateway
or
Transit Gateway

↓

VPC Route Table

↓

Subnet

↓

Application Load Balancer

↓

EC2 / ECS / EKS

↓

Application
```

Every device in this path must forward traffic correctly.

---

# 73. VPN Troubleshooting Flow

Whenever a VPN issue occurs, follow this order:

```
Tunnel Status

↓

BGP Status

↓

Route Tables

↓

Security Groups

↓

Network ACL

↓

Firewall

↓

Application
```

Never start troubleshooting at the EC2 instance.

Start from the tunnel.

---

# 74. Tunnel Status

Check whether:

```
Tunnel 1

UP

Tunnel 2

UP
```

Possible states:

```
UP

DOWN

ESTABLISHING
```

If both tunnels are DOWN:

- Check Customer Gateway
- Check Internet connectivity
- Check firewall
- Check IPSec configuration

---

# 75. Verify BGP

If using dynamic routing:

Verify:

```
BGP State

Established
```

If not established:

Possible reasons:

- Wrong ASN
- Firewall blocking TCP 179
- Incorrect tunnel configuration
- Customer Gateway issue

---

# 76. Verify Route Tables

Check AWS Route Table:

Example:

```
Destination

192.168.10.0/24

↓

VGW

or

Transit Gateway
```

Check On-Prem Router:

```
Destination

10.0.0.0/16

↓

VPN Tunnel
```

Missing routes are one of the most common causes of connectivity failures.

---

# 77. Security Groups

Even if routing works:

Security Groups may block traffic.

Example:

```
HTTPS

443

Denied
```

Result:

VPN works

Application unreachable

---

# 78. Network ACL

Remember:

Network ACLs are

Stateless.

Both:

Inbound

and

Outbound

must allow traffic.

---

# 79. Firewall Verification

Corporate firewalls must allow VPN traffic.

Important ports/protocols include:

- UDP 500 (IKE)
- UDP 4500 (NAT Traversal)
- ESP (IP Protocol 50) where applicable
- TCP 179 (BGP, if dynamic routing is used)

---

# 80. CloudWatch Monitoring

Useful metrics:

- TunnelState
- TunnelDataIn
- TunnelDataOut
- PacketDropCount
- TunnelStatus

Create CloudWatch alarms for:

- Tunnel Down
- High packet loss
- Tunnel flapping

---

# 81. VPC Flow Logs

Flow Logs help determine whether traffic reaches the VPC.

Example:

```
ACCEPT

Traffic reached EC2.
```

```
REJECT

Traffic blocked.
```

Very useful for troubleshooting Security Groups and Network ACLs.

---

# 82. AWS CLI Commands

## Describe VPN Connections

```bash
aws ec2 describe-vpn-connections
```

---

## Describe Customer Gateways

```bash
aws ec2 describe-customer-gateways
```

---

## Describe Virtual Private Gateways

```bash
aws ec2 describe-vpn-gateways
```

---

## Describe Transit Gateway Attachments

```bash
aws ec2 describe-transit-gateway-attachments
```

---

# 83. Terraform Example

## Customer Gateway

```hcl
resource "aws_customer_gateway" "corp" {

  bgp_asn    = 65000

  ip_address = "203.0.113.10"

  type       = "ipsec.1"

  tags = {

    Name = "Corporate-CGW"

  }

}
```

---

## Virtual Private Gateway

```hcl
resource "aws_vpn_gateway" "vgw" {

  vpc_id = aws_vpc.main.id

}
```

---

## VPN Connection

```hcl
resource "aws_vpn_connection" "vpn" {

  customer_gateway_id = aws_customer_gateway.corp.id

  vpn_gateway_id      = aws_vpn_gateway.vgw.id

  type                = "ipsec.1"

  static_routes_only  = false

}
```

---

# 84. Production Best Practices

✔ Prefer BGP over static routing.

✔ Use Transit Gateway for multiple VPCs.

✔ Keep CIDR blocks non-overlapping.

✔ Enable CloudWatch monitoring.

✔ Enable VPC Flow Logs.

✔ Rotate Pre-Shared Keys regularly.

✔ Test tunnel failover.

✔ Document routing.

✔ Use Infrastructure as Code (Terraform or CloudFormation).

---

# 85. Common Mistakes

❌ Wrong Customer Gateway IP.

❌ Wrong Pre-Shared Key.

❌ Firewall blocking UDP 500.

❌ Firewall blocking UDP 4500.

❌ Incorrect ASN.

❌ Missing routes.

❌ Overlapping CIDRs.

❌ Security Groups blocking traffic.

❌ Network ACL blocking traffic.

❌ Assuming "Tunnel UP" means "Application works."

---

# 86. Real Production Scenario 1

## Problem

VPN Tunnel is UP.

Application is unreachable.

### Investigation

Tunnel

✔

BGP

✔

Security Group

❌

Port 443 blocked.

### Root Cause

Application traffic denied by Security Group.

---

# 87. Real Production Scenario 2

## Problem

Both tunnels DOWN.

### Investigation

Firewall changed.

UDP 500 blocked.

### Solution

Allow:

- UDP 500
- UDP 4500
- ESP (if required)

VPN immediately recovers.

---

# 88. Real Production Scenario 3

## Problem

Tunnel UP.

Ping fails.

### Investigation

AWS Route Table missing:

```
192.168.1.0/24

↓

VGW
```

### Solution

Add route.

Traffic starts working.

---

# 89. Real Production Scenario 4

## Problem

VPN recreated.

Tunnel never establishes.

### Investigation

Customer Gateway public IP changed.

AWS still points to the old IP.

### Solution

Update the Customer Gateway configuration or create a new Customer Gateway with the correct public IP.

---

# 90. Decision Matrix

| Requirement | Recommended Solution |
|-------------|----------------------|
| Connect one VPC | VPN + VGW |
| Connect multiple VPCs | VPN + TGW |
| Mission Critical | Direct Connect |
| Backup Connectivity | VPN |
| Hybrid Cloud | TGW + VPN |
| Multiple Accounts | TGW + AWS RAM |

---

# 91. Advanced Interview Questions

## Question 16

What is the difference between Customer Gateway and Virtual Private Gateway?

### Answer

Customer Gateway represents the on-premises VPN device.

Virtual Private Gateway is the AWS VPN endpoint attached to a VPC.

---

## Question 17

Why does AWS create two VPN tunnels?

### Answer

To provide high availability.

If one tunnel fails, traffic can continue through the second tunnel.

---

## Question 18

Can Site-to-Site VPN work without BGP?

### Answer

Yes.

Static routing can be used, but BGP is preferred because it automatically exchanges routes and supports easier failover.

---

## Question 19

What ports are required for AWS Site-to-Site VPN?

### Answer

- UDP 500 (IKE)
- UDP 4500 (NAT Traversal)
- TCP 179 (BGP, if dynamic routing is used)
- ESP (IP Protocol 50), depending on the environment and NAT usage

---

## Question 20

Can VPN traffic be inspected?

### Answer

Yes.

Traffic can be routed through an Inspection VPC using a Transit Gateway, where centralized firewalls or IDS/IPS appliances inspect it.

---

## Question 21

Why would you choose Transit Gateway instead of Virtual Private Gateway?

### Answer

Transit Gateway supports multiple VPCs, multiple AWS accounts, and centralized routing, making it the preferred choice for enterprise environments.

---

## Question 22

What happens if one VPN tunnel fails?

### Answer

Traffic automatically switches to the second tunnel if routing and failover are configured correctly.

---

## Question 23

Can Direct Connect replace VPN?

### Answer

It depends.

Direct Connect provides a dedicated private connection but does not encrypt traffic by default. Many organizations use Direct Connect for primary connectivity and VPN as an encrypted backup path.

---

## Question 24

How do you troubleshoot a VPN that is UP but not passing traffic?

### Answer

Check:

- BGP status
- Route tables
- Security Groups
- Network ACLs
- Firewall rules
- Application listening ports
- VPC Flow Logs

---

## Question 25

What are the most common reasons Site-to-Site VPN fails?

### Answer

- Incorrect Pre-Shared Key
- Wrong Customer Gateway IP
- Firewall blocking VPN traffic
- Incorrect BGP configuration
- Missing routes
- Overlapping CIDRs
- Security Group or Network ACL restrictions

---

# 92. Hands-on Labs

## Lab 16

Create a complete VPN using Terraform.

---

## Lab 17

Configure BGP between the Customer Gateway and AWS.

---

## Lab 18

Simulate Tunnel 1 failure and verify failover.

---

## Lab 19

Enable CloudWatch alarms for tunnel health.

---

## Lab 20

Troubleshooting Exercise:

Break the VPN by:

- Removing a route
- Blocking UDP 500
- Changing the Pre-Shared Key
- Blocking TCP 179

Identify and resolve each issue.

---

# 93. One-Page Revision

```
Corporate Office

↓

Customer Gateway

↓

Tunnel 1

Tunnel 2

↓

VGW / TGW

↓

Route Table

↓

Application
```

Remember:

- Site-to-Site VPN uses IPSec.
- AWS creates two tunnels for redundancy.
- BGP enables automatic route exchange.
- Always verify routing on both AWS and on-premises.
- Monitor tunnel health using CloudWatch.
- Use Transit Gateway for scalable hybrid architectures.

---

# Chapter Summary

In this chapter, you learned:

- Site-to-Site VPN fundamentals
- IPSec and IKE
- Customer Gateway and Virtual Private Gateway
- Static vs Dynamic Routing
- BGP and Route Propagation
- Tunnel redundancy and failover
- Transit Gateway integration
- Hybrid cloud architectures
- Direct Connect comparison
- Monitoring and troubleshooting
- Terraform and AWS CLI examples
- Production best practices
- Advanced interview questions

You should now be able to design, deploy, troubleshoot, and explain AWS Site-to-Site VPN solutions confidently in production and interview scenarios.

---

# Think Like a Production Engineer

A Site-to-Site VPN is not just a tunnel—it is a complete hybrid networking solution.

When designing or troubleshooting:

1. Verify tunnel health first.
2. Confirm BGP or static routing.
3. Check AWS and on-premises route tables.
4. Validate Security Groups and Network ACLs.
5. Monitor with CloudWatch and Flow Logs.
6. Test failover regularly instead of assuming redundancy works.

Enterprise engineers don't stop when the tunnel shows **UP**—they validate that **applications, routing, security, and monitoring** all work together.

# End of Chapter 19 – Site-to-Site VPN
