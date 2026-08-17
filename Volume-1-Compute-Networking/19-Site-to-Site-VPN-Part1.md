# Site-to-Site VPN

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 19
>
> Site-to-Site VPN – Part 1

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Site-to-Site VPN
- Explain why enterprises use VPNs
- Understand IPSec
- Understand IKE Phase 1 and Phase 2
- Configure Customer Gateway (CGW)
- Configure Virtual Private Gateway (VGW)
- Understand VPN packet flow
- Explain encryption and authentication

---

# 1. Why Do We Need Site-to-Site VPN?

Very few organizations move everything to AWS.

A typical enterprise has:

- Corporate Data Center
- VMware Cluster
- Oracle Database
- SAP Servers
- Active Directory
- Internal Applications

These systems still need to communicate securely with AWS resources.

Example:

```
Corporate Data Center

192.168.1.0/24

        │

        ?

        │

AWS VPC

10.0.0.0/16
```

The challenge is providing **private and secure communication** over the Internet.

AWS solves this using a **Site-to-Site VPN**.

---

# 2. What is Site-to-Site VPN?

A Site-to-Site VPN is an **encrypted IPSec tunnel** between:

- Your on-premises network
- An AWS Virtual Private Gateway (VGW) or Transit Gateway (TGW)

It allows resources to communicate securely as though they were on the same private network.

---

# 3. Real Production Example

```
                 Corporate Office

              192.168.10.0/24

                      │

               IPSec VPN Tunnel

                      │

                 AWS VPC

               10.0.0.0/16
```

Employees access applications hosted in AWS without exposing them to the public Internet.

---

# 4. What Problems Does VPN Solve?

Without VPN:

- Traffic crosses the Internet unencrypted.
- Data can be intercepted.
- Compliance requirements may not be met.

With VPN:

- Data is encrypted.
- Data integrity is protected.
- Authentication is performed.
- Communication is secure.

---

# 5. VPN Architecture

```
+---------------------------+

| Corporate Data Center     |

| 192.168.1.0/24            |

+-------------+-------------+

              │

      Customer Gateway

              │

      ===================

        IPSec Tunnel

      ===================

              │

     Virtual Private Gateway

              │

+-------------+-------------+

| AWS VPC                   |

|10.0.0.0/16                |

+---------------------------+
```

---

# 6. Components of AWS Site-to-Site VPN

A Site-to-Site VPN consists of:

- Customer Gateway (CGW)
- Virtual Private Gateway (VGW) or Transit Gateway (TGW)
- IPSec Tunnel
- Route Tables
- BGP or Static Routes

---

# 7. Customer Gateway (CGW)

A Customer Gateway represents your on-premises VPN device.

Examples:

- Cisco Router
- Fortinet Firewall
- Palo Alto Firewall
- Juniper SRX
- Sophos Firewall

AWS stores information such as:

- Public IP Address
- Routing type
- BGP ASN (if dynamic routing is used)

---

# 8. Virtual Private Gateway (VGW)

A Virtual Private Gateway is the AWS endpoint for a Site-to-Site VPN when connecting to a single VPC.

```
On-Prem

↓

Customer Gateway

↓

VPN Tunnel

↓

VGW

↓

VPC
```

The VGW is attached to the VPC.

---

# 9. Transit Gateway vs VGW

### VGW

- Attached to one VPC.
- Simpler environments.
- Smaller deployments.

### TGW

- Connects multiple VPCs.
- Enterprise environments.
- Centralized routing.
- Hybrid networking.

---

# 10. IPSec (Internet Protocol Security)

IPSec is the protocol suite that secures VPN traffic.

It provides:

- Encryption
- Authentication
- Integrity
- Anti-replay protection

Without IPSec, VPN traffic would be readable on the public Internet.

---

# 11. IPSec Services

### Confidentiality

Encrypts data so unauthorized users cannot read it.

---

### Integrity

Ensures the packet has not been modified during transit.

---

### Authentication

Verifies the identity of the VPN peers.

---

### Anti-Replay

Prevents attackers from capturing and retransmitting packets.

---

# 12. Tunnel Mode vs Transport Mode

### Transport Mode

Only the packet payload is encrypted.

```
IP Header

↓

Visible

Payload

↓

Encrypted
```

Used less commonly for AWS Site-to-Site VPN.

---

### Tunnel Mode

The entire original IP packet is encapsulated and encrypted.

```
New IP Header

↓

Original Packet

↓

Encrypted
```

AWS Site-to-Site VPN uses **Tunnel Mode**.

---

# 13. Internet Key Exchange (IKE)

Before encrypted traffic can flow, both VPN endpoints must agree on:

- Encryption algorithm
- Authentication method
- Keys
- Security parameters

This negotiation is performed using **IKE**.

---

# 14. IKE Phase 1

Purpose:

Create a secure management channel.

During Phase 1:

- Peers authenticate each other.
- Encryption algorithms are negotiated.
- Diffie-Hellman keys are exchanged.
- A secure IKE Security Association (SA) is established.

Think of it as both devices introducing themselves and agreeing on the rules before exchanging sensitive information.

---

# 15. IKE Phase 2

Purpose:

Create the IPSec tunnel used to carry actual application traffic.

During Phase 2:

- IPSec Security Associations are negotiated.
- Data encryption keys are generated.
- The encrypted tunnel becomes operational.

---

# 16. VPN Packet Flow

```
Application

↓

TCP/IP Packet

↓

IPSec Encryption

↓

Internet

↓

AWS VPN Endpoint

↓

IPSec Decryption

↓

Destination EC2
```

---

# 17. Encryption Algorithms

Common encryption algorithms:

- AES-128
- AES-192
- AES-256

AWS recommends modern, secure algorithms such as AES-256 where supported.

---

# 18. Authentication Algorithms

Common algorithms include:

- SHA-1 (legacy)
- SHA-256
- SHA-384
- SHA-512

Stronger SHA-2 family algorithms are generally preferred.

---

# 19. Diffie-Hellman (DH)

Diffie-Hellman enables both VPN endpoints to establish a shared secret key over an untrusted network.

This shared secret is then used to derive encryption keys.

---

# 20. Security Association (SA)

A Security Association contains:

- Encryption algorithm
- Authentication algorithm
- Lifetime
- Keys
- Tunnel parameters

Each VPN tunnel maintains Security Associations for secure communication.

---

# 21. Best Practices

- Use strong encryption (AES-256 where supported).
- Use SHA-2 authentication algorithms.
- Use IKEv2 where supported.
- Rotate pre-shared keys regularly.
- Monitor tunnel health.
- Document VPN configurations.

---

# 22. Common Mistakes

❌ Incorrect Customer Gateway IP.

❌ Mismatched encryption settings.

❌ Wrong pre-shared key.

❌ Incorrect routing.

❌ Firewall blocking UDP 500 or UDP 4500 (used by IKE/NAT-T).

❌ Assuming VPN traffic is automatically routed without route configuration.

---

# 23. Interview Questions

## Question 1

What is AWS Site-to-Site VPN?

### Answer

AWS Site-to-Site VPN is a managed IPSec VPN service that securely connects an on-premises network to AWS over the Internet using encrypted tunnels.

---

## Question 2

What is a Customer Gateway?

### Answer

A Customer Gateway is the on-premises VPN device or its AWS representation. It contains configuration such as the public IP address and routing information.

---

## Question 3

What is a Virtual Private Gateway?

### Answer

A Virtual Private Gateway is the AWS-side VPN endpoint attached to a VPC. It terminates VPN tunnels for that VPC.

---

## Question 4

What is IPSec?

### Answer

IPSec is a suite of protocols that provides encryption, authentication, integrity, and anti-replay protection for IP traffic.

---

## Question 5

What is the difference between IKE Phase 1 and Phase 2?

### Answer

Phase 1 establishes a secure management channel and negotiates security parameters. Phase 2 creates the IPSec Security Associations that encrypt application traffic.

---

# 24. Hands-on Labs

## Lab 1

Create:

- VPC
- Virtual Private Gateway

---

## Lab 2

Create a Customer Gateway using a sample public IP.

---

## Lab 3

Create a Site-to-Site VPN connection between the Customer Gateway and Virtual Private Gateway.

---

## Lab 4

Download the vendor-specific VPN configuration generated by AWS.

Review the parameters for a Cisco or Fortinet device.

---

## Lab 5

Identify:

- IKE settings
- IPSec settings
- Tunnel IP addresses
- Pre-shared keys
- Routing configuration

---

# 25. One-Page Revision

```
On-Prem Router

↓

Customer Gateway

↓

IKE Phase 1

↓

IKE Phase 2

↓

IPSec Tunnel

↓

Virtual Private Gateway

↓

VPC
```

Remember:

- Site-to-Site VPN uses IPSec.
- IKE negotiates secure parameters.
- Tunnel Mode encrypts the original IP packet.
- CGW = On-premises endpoint.
- VGW = AWS endpoint for a VPC.
- Use strong encryption and authentication algorithms.

---

# Think Like a Production Engineer

A Site-to-Site VPN is more than "a secure tunnel."

It is a complete system involving:

1. Correct routing.
2. Matching cryptographic settings.
3. Healthy tunnel status.
4. Proper firewall rules.
5. Monitoring and failover.

In production, VPN issues are often caused by configuration mismatches rather than AWS itself. Always verify both AWS and on-premises configurations when troubleshooting.

# End of Part 1
