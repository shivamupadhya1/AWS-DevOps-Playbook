# AWS Client VPN

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 21
>
> AWS Client VPN – Part 3 (Architecture, Routing, High Availability & Production Design)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Design production-ready Client VPN architectures
- Understand multi-AZ Client VPN deployment
- Learn Client VPN routing
- Integrate Client VPN with Transit Gateway
- Access On-Premises resources
- Design Hybrid Cloud connectivity
- Learn high availability and failover
- Understand enterprise production scenarios

---

# 44. Multi-AZ Client VPN Architecture

AWS Client VPN is a **Regional service**, but to make it highly available, you associate it with **subnets in multiple Availability Zones (AZs).**

Example:

```
                     AWS Region

---------------------------------------------------------

              Client VPN Endpoint

                 /           \

                /             \

        Target Subnet A    Target Subnet B

            AZ-A              AZ-B

               │                 │

             EC2-A            EC2-B
```

If one Availability Zone becomes unavailable, users can continue accessing resources through the remaining associated subnet(s).

---

# 45. High Availability

AWS recommends:

- Associate Client VPN with multiple subnets
- Deploy applications across multiple AZs
- Use Application Load Balancers
- Enable Auto Scaling where applicable

Example:

```
Laptop

↓

Client VPN

↓

AZ-A

↓

EC2

OR

↓

AZ-B

↓

EC2
```

---

# 46. Route Table Design

The Client VPN endpoint has its own route table.

Example:

```
Destination

10.0.0.0/16

↓

Target Network
```

If the VPC contains multiple networks:

```
10.0.1.0/24

Applications

10.0.2.0/24

Databases

10.0.3.0/24

Monitoring
```

Routes determine how client traffic reaches each destination.

---

# 47. Route Propagation

Client VPN does **not** automatically learn every route.

You must add the required routes to the Client VPN endpoint.

Example:

```
Client VPN

↓

10.0.1.0/24

↓

Application

Client VPN

↓

10.0.2.0/24

↓

Database
```

Without these routes, traffic will not reach the destination.

---

# 48. Full Packet Flow

```
Developer Laptop

↓

AWS VPN Client

↓

Internet

↓

Client VPN Endpoint

↓

Authentication

↓

Authorization Rule

↓

Client VPN Route Table

↓

Target Network Association

↓

VPC Route Table

↓

Private Subnet

↓

EC2
```

This is a very common interview diagram.

---

# 49. Client VPN + Transit Gateway

Client VPN can provide access to multiple VPCs by integrating with a **Transit Gateway (TGW).**

```
Laptop

↓

Client VPN

↓

Transit Gateway

↓

Production VPC

↓

Development VPC

↓

Shared Services VPC
```

Benefits:

- One VPN endpoint
- Centralized routing
- Easier management
- Better scalability

---

# 50. Client VPN + Site-to-Site VPN

Many enterprises combine both services.

```
Developer Laptop

↓

Client VPN

↓

AWS VPC

↓

Transit Gateway

↓

Site-to-Site VPN

↓

Corporate Data Center
```

Now remote users can access:

- AWS resources
- On-premises servers

through one VPN connection.

---

# 51. Client VPN + Direct Connect

Some organizations also have Direct Connect.

```
Laptop

↓

Client VPN

↓

Transit Gateway

↓

Direct Connect

↓

Corporate Office
```

This allows remote employees to securely access on-premises applications through AWS networking.

---

# 52. Hybrid Cloud Architecture

```
Remote Employee

↓

Client VPN

↓

Transit Gateway

↓

Production VPC

↓

Development VPC

↓

Shared Services

↓

Site-to-Site VPN

↓

Corporate Data Center
```

This is a common enterprise architecture.

---

# 53. Shared Services Access

Shared Services VPC

```
GitLab

↓

Jenkins

↓

Nexus

↓

Prometheus

↓

Grafana

↓

Active Directory
```

Developers connect through Client VPN to access these internal services securely.

---

# 54. Bastion Host Replacement

Older architecture:

```
Laptop

↓

Public Bastion Host

↓

Private EC2
```

Modern architecture:

```
Laptop

↓

AWS Client VPN

↓

Private EC2
```

Advantages:

- No public SSH bastion
- Smaller attack surface
- Better auditing
- Simpler access management

---

# 55. Remote Database Access

Example:

```
Developer

↓

Client VPN

↓

Private RDS
```

Without Client VPN:

The database remains inaccessible from the Internet.

This improves security.

---

# 56. DevOps Production Example

A DevOps engineer needs access to:

- Jenkins
- Nexus
- GitLab
- Kubernetes API Server
- Monitoring tools

Architecture:

```
Laptop

↓

Client VPN

↓

Transit Gateway

↓

Shared Services VPC

↓

Jenkins

GitLab

Nexus

Prometheus

Grafana
```

No service requires a public IP address.

---

# 57. Monitoring

Monitor:

- Client connections
- Authentication failures
- Authorization failures
- Connection duration
- CloudWatch Logs
- VPN endpoint health

Useful metrics include:

- Active client connections
- Connection attempts
- Failed authentication events (via logs)

---

# 58. Best Practices

✔ Associate Client VPN with multiple AZs.

✔ Integrate with Transit Gateway for multiple VPCs.

✔ Use SAML + MFA.

✔ Keep applications in private subnets.

✔ Use least-privilege authorization.

✔ Monitor CloudWatch Logs.

✔ Audit VPN access regularly.

---

# 59. Common Mistakes

❌ Associating the endpoint with only one subnet.

❌ Forgetting Client VPN routes.

❌ No authorization rules.

❌ Publicly exposing Jenkins or GitLab instead of using VPN.

❌ No MFA.

❌ Using overly broad network access.

---

# 60. Production Scenario 1

## Problem

Developers connect successfully.

Cannot reach the Development VPC.

Investigation:

Authentication ✔

Authorization ✔

Client VPN Route ❌ Missing

Solution:

Add the required route to the Client VPN endpoint.

---

# 61. Production Scenario 2

## Problem

Users can access AWS resources.

Cannot access on-premises applications.

Root Cause:

Transit Gateway or Site-to-Site VPN routing is incomplete.

---

# 62. Production Scenario 3

## Problem

One Availability Zone becomes unavailable.

Expected Behavior:

Users continue connecting through the Client VPN endpoint because it is associated with another subnet in a different AZ.

---

# 63. Production Scenario 4

## Problem

GitLab is publicly accessible.

Security team rejects the design.

Solution:

Move GitLab into a private subnet and provide access through Client VPN.

---

# 64. Interview Questions

## Question 11

Can Client VPN access multiple VPCs?

### Answer

Yes.

By integrating Client VPN with a Transit Gateway, users can access resources across multiple attached VPCs.

---

## Question 12

Can Client VPN access on-premises resources?

### Answer

Yes.

If routing exists through Transit Gateway, Site-to-Site VPN, or Direct Connect, Client VPN users can access on-premises resources.

---

## Question 13

Why is Client VPN preferred over a Bastion Host?

### Answer

Client VPN provides secure, authenticated access to private resources without exposing SSH or RDP services to the Internet.

---

## Question 14

How do you make Client VPN highly available?

### Answer

Associate the Client VPN endpoint with subnets in multiple Availability Zones and deploy applications across multiple AZs.

---

## Question 15

Can Client VPN connect to private RDS databases?

### Answer

Yes.

If routing, authorization rules, Security Groups, and Network ACLs allow access, users can securely connect to private RDS instances.

---

# 65. Hands-on Labs

## Lab 11

Design a Client VPN architecture with:

- Two AZs
- Private EC2
- Application Load Balancer

---

## Lab 12

Integrate Client VPN with Transit Gateway and three VPCs.

---

## Lab 13

Design a remote developer environment providing secure access to:

- Jenkins
- GitLab
- Nexus
- Kubernetes

---

## Lab 14

Create a hybrid cloud architecture where Client VPN users access both AWS and on-premises applications.

---

## Lab 15

Replace a Bastion Host architecture with Client VPN and document the security improvements.

---

# 66. One-Page Revision

```
Laptop

↓

AWS Client VPN

↓

Authentication

↓

Authorization

↓

Route Table

↓

Transit Gateway

↓

Production VPC

↓

Development VPC

↓

Shared Services

↓

Site-to-Site VPN

↓

Corporate Data Center
```

Remember:

- Associate Client VPN with multiple AZs.
- Add required Client VPN routes.
- Transit Gateway enables access to multiple VPCs.
- Client VPN can also provide access to on-premises resources.
- Replace Bastion Hosts with Client VPN where appropriate.
- Keep applications in private subnets.

---

# Think Like a Production Engineer

In modern enterprises, remote access should never rely on publicly exposed servers.

Instead:

1. Use Client VPN as the secure entry point.
2. Integrate it with Transit Gateway for centralized networking.
3. Keep applications private.
4. Enable MFA and centralized identity management.
5. Audit access using CloudWatch Logs.
6. Design for high availability across multiple Availability Zones.

A production-grade remote access solution is secure, scalable, auditable, and easy to manage.

# End of Part 3
