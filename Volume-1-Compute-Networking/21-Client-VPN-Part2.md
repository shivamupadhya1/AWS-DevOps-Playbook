# AWS Client VPN

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 21
>
> AWS Client VPN – Part 2 (Authentication, Authorization, Routing & Security)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Client VPN authentication methods
- Learn certificate-based authentication
- Understand Active Directory authentication
- Learn SAML authentication
- Understand authorization rules
- Configure routing
- Understand security flow
- Troubleshoot authentication issues

---

# 18. Authentication in Client VPN

Authentication answers the question:

> **"Who is trying to connect?"**

AWS Client VPN supports three authentication methods:

```
Client VPN Endpoint

        │

 ┌──────┼───────────┐

 │      │           │

Certificate    Active Directory    SAML

Authentication Authentication      Authentication
```

A user cannot access the VPN until authentication succeeds.

---

# 19. Mutual Certificate Authentication

This is the simplest authentication method.

AWS verifies the client certificate before allowing access.

```
Laptop

↓

Client Certificate

↓

Client VPN Endpoint

↓

Certificate Verified

↓

Connection Allowed
```

Both client and server trust certificates issued by the same Certificate Authority (CA).

---

# 20. How Mutual TLS (mTLS) Works

```
Laptop

↓

Sends Certificate

↓

AWS Client VPN

↓

Verifies Certificate

↓

Trusted CA

↓

Connection Established
```

Advantages:

- Strong authentication
- No password required
- Good for administrators or small teams

Disadvantages:

- Certificate distribution
- Certificate renewal
- Revocation management

---

# 21. Active Directory Authentication

Large enterprises commonly authenticate users through Microsoft Active Directory.

Supported options include:

- AWS Managed Microsoft AD
- Self-managed Microsoft AD connected to AWS

Example:

```
Employee

↓

Username + Password

↓

AWS Managed Microsoft AD

↓

Authentication Success

↓

Client VPN

↓

AWS Resources
```

---

# 22. Benefits of Active Directory

- Existing corporate accounts
- Centralized password management
- Group-based access
- Easy onboarding/offboarding
- Familiar login experience

---

# 23. SAML Authentication

Modern organizations often use Single Sign-On (SSO).

AWS Client VPN supports SAML 2.0 identity providers.

Examples:

- Okta
- Microsoft Entra ID (Azure AD)
- Ping Identity
- OneLogin
- Keycloak

```
User

↓

Identity Provider

↓

SAML Assertion

↓

AWS Client VPN

↓

Connection Allowed
```

---

# 24. Benefits of SAML

- Single Sign-On
- MFA support
- Centralized identity
- Corporate security policies
- Easy integration with enterprise IdPs

---

# 25. Authentication Comparison

| Method | Best For |
|----------|----------|
| Certificate | Small teams, administrators, labs |
| Active Directory | Enterprises using Microsoft AD |
| SAML | Organizations with SSO and MFA |

---

# 26. Authorization Rules

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to access?**

Example:

```
Developer

↓

VPN Connected

↓

Can Access

10.0.1.0/24

↓

Cannot Access

10.0.5.0/24
```

---

# 27. Authorization Rule Example

Suppose the VPC contains:

```
10.0.1.0/24

Application Servers

10.0.2.0/24

Database Servers

10.0.3.0/24

Management Servers
```

Developers may be allowed:

```
✔ Application Servers

✔ Test Servers

✖ Databases

✖ Management
```

Authorization rules enforce this.

---

# 28. Target Network Association

A Client VPN endpoint does **not** automatically know where to send traffic.

You must associate it with one or more target subnets.

```
Client VPN Endpoint

↓

Target Network Association

↓

Private Subnet

↓

EC2
```

Without a target network association, clients can connect but cannot reach VPC resources.

---

# 29. Client VPN Route Table

The Client VPN endpoint has its own route table.

Example:

```
Destination

10.0.0.0/16

↓

Target Subnet
```

When a user sends traffic to:

```
10.0.1.25
```

The endpoint checks its route table and forwards traffic to the associated subnet.

---

# 30. Packet Flow

```
Developer Laptop

↓

VPN Client

↓

Internet

↓

Client VPN Endpoint

↓

Authentication

↓

Authorization

↓

Route Table

↓

Target Subnet

↓

EC2
```

---

# 31. Security Groups

A Security Group is associated with the Client VPN endpoint.

Example:

```
Allow

HTTPS

443

Allow

SSH

22
```

Even if authentication succeeds, Security Groups can still block traffic.

---

# 32. Network ACL

After traffic enters the VPC:

```
Client VPN

↓

Subnet

↓

Network ACL

↓

EC2
```

If the Network ACL blocks traffic:

The VPN connection succeeds.

The application remains unreachable.

---

# 33. Split Tunnel

Normally:

```
Laptop

↓

All Internet Traffic

↓

VPN

↓

AWS
```

This is called **Full Tunnel**.

---

With **Split Tunnel**:

```
AWS Traffic

↓

VPN

Internet Browsing

↓

Local ISP
```

Benefits:

- Lower VPN bandwidth usage
- Better user experience
- Reduced latency for general Internet traffic

---

# 34. Full Tunnel vs Split Tunnel

| Full Tunnel | Split Tunnel |
|-------------|--------------|
| All traffic goes through VPN | Only AWS traffic goes through VPN |
| Higher bandwidth usage | Lower bandwidth usage |
| Centralized inspection | Better Internet performance |
| Stronger control | More efficient for remote users |

---

# 35. Logging

Enable connection logging to:

- Audit user logins
- Troubleshoot authentication
- Investigate security incidents

Logs are typically sent to **Amazon CloudWatch Logs**.

---

# 36. Best Practices

✔ Use SAML with MFA for enterprise users.

✔ Apply least-privilege authorization rules.

✔ Enable CloudWatch logging.

✔ Enable split tunnel only if it aligns with your organization's security policy.

✔ Restrict Security Groups.

✔ Review user access regularly.

---

# 37. Common Mistakes

❌ No authorization rules.

❌ No target network association.

❌ Forgetting Client VPN routes.

❌ Security Group blocks VPN traffic.

❌ Network ACL blocks return traffic.

❌ Using certificate authentication when centralized identity is required.

---

# 38. Production Scenario 1

### Problem

Users connect successfully.

Cannot access EC2.

Investigation:

Authentication ✔

Authorization ✔

Security Group ❌

Port 22 blocked.

---

# 39. Production Scenario 2

### Problem

VPN authentication fails.

Root Cause:

Active Directory unavailable.

Solution:

Restore directory connectivity and verify directory health.

---

# 40. Production Scenario 3

### Problem

Developers can access production databases.

Root Cause:

Authorization rule allowed:

```
10.0.0.0/16
```

instead of only the required application subnet.

---

# 41. Interview Questions

## Question 6

What authentication methods are supported by AWS Client VPN?

### Answer

- Mutual certificate authentication
- Active Directory authentication
- SAML 2.0 authentication

---

## Question 7

What is the difference between authentication and authorization?

### Answer

Authentication verifies the user's identity.

Authorization determines which resources the authenticated user can access.

---

## Question 8

Why is a Target Network Association required?

### Answer

It associates the Client VPN endpoint with one or more VPC subnets so traffic can be forwarded into the VPC.

---

## Question 9

What is Split Tunnel?

### Answer

Split Tunnel sends only traffic destined for configured AWS networks through the VPN, while other Internet traffic uses the user's local Internet connection.

---

## Question 10

What happens if there is no authorization rule?

### Answer

The user may successfully authenticate and establish the VPN connection but will not be able to access the protected network because access is not authorized.

---

# 42. Hands-on Labs

## Lab 6

Configure:

- Certificate authentication
- Connect using AWS VPN Client

---

## Lab 7

Integrate Client VPN with AWS Managed Microsoft AD.

---

## Lab 8

Configure SAML authentication using an enterprise Identity Provider.

---

## Lab 9

Create authorization rules allowing developers to access only application servers.

---

## Lab 10

Enable split tunnel and compare traffic flow with full tunnel mode.

---

# 43. One-Page Revision

```
Laptop

↓

VPN Client

↓

Authentication

↓

Authorization

↓

Client VPN Route Table

↓

Target Network

↓

Security Group

↓

Network ACL

↓

EC2
```

Remember:

- Authentication verifies identity.
- Authorization controls access.
- Three authentication methods:
  - Certificate
  - Active Directory
  - SAML
- Target Network Association is mandatory.
- Split Tunnel routes only AWS traffic through the VPN.
- CloudWatch Logs help with auditing and troubleshooting.

---

# Think Like a Production Engineer

A secure Client VPN deployment is not just about allowing users to connect—it is about ensuring they can access **only** the resources they need.

When designing Client VPN:

1. Use centralized authentication (SAML or Active Directory) for enterprise environments.
2. Enable MFA whenever possible.
3. Apply least-privilege authorization rules.
4. Associate the endpoint with the correct target subnets.
5. Verify Security Groups and Network ACLs during troubleshooting.
6. Enable logging and regularly audit VPN access.

Production security is achieved by combining **authentication, authorization, routing, and monitoring**.

# End of Part 2
