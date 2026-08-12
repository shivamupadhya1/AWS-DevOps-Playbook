# Transit Gateway

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 16
>
> Transit Gateway – Part 1

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Transit Gateway
- Explain why AWS introduced TGW
- Design Hub-and-Spoke architecture
- Configure VPC Attachments
- Understand TGW Route Tables
- Compare TGW with VPC Peering
- Design enterprise networks
- Answer production interview questions

---

# 1. Why Was Transit Gateway Introduced?

Suppose your company has:

```
Production VPC

Development VPC

Testing VPC

Shared Services VPC

Monitoring VPC

Security VPC

Networking VPC
```

Should every VPC peer with every other VPC?

No.

The number of peering connections grows rapidly and becomes difficult to manage.

AWS introduced:

```
Transit Gateway
```

to simplify large-scale networking.

---

# 2. The Problem with VPC Peering

Suppose you have four VPCs.

```
A

B

C

D
```

To allow every VPC to communicate with every other VPC, you need multiple peering connections.

```
A ↔ B

A ↔ C

A ↔ D

B ↔ C

B ↔ D

C ↔ D
```

As the number of VPCs increases, the number of connections grows quickly, making management difficult.

---

# 3. What is Transit Gateway?

Transit Gateway (TGW) is a **regional network hub** that connects multiple VPCs, VPNs, and Direct Connect gateways through a single centralized routing service.

Think of it as a central router inside AWS.

---

# 4. Real-Life Analogy

Imagine a city.

Without a central highway:

Every colony has to build roads to every other colony.

```
A ↔ B

A ↔ C

A ↔ D
```

This is similar to VPC Peering.

With a highway:

```
      Highway

A → Highway

B → Highway

C → Highway

D → Highway
```

This is Transit Gateway.

---

# 5. Hub-and-Spoke Architecture

```
                 +-------------------+
                 | Transit Gateway   |
                 +---------+---------+
                           |
       -----------------------------------------
       |          |          |         |        |
       |          |          |         |        |
 +-----+---+ +----+---+ +----+---+ +---+----+ +----+---+
 | Prod VPC| | Dev VPC| | Test VPC| | Shared | | Logging |
 |10.0.0.0 | |10.1.0.0| |10.2.0.0 | |Services| |10.4.0.0 |
 +---------+ +--------+ +---------+ +--------+ +---------+
```

Every VPC connects only to the Transit Gateway.

---

# 6. Advantages of Transit Gateway

- Centralized routing
- Simpler architecture
- Scales to thousands of VPC attachments (subject to AWS quotas)
- Supports hybrid networking
- Supports cross-account connectivity
- Supports Direct Connect
- Supports Site-to-Site VPN

---

# 7. Components of Transit Gateway

A Transit Gateway environment consists of:

- Transit Gateway
- Attachments
- Route Tables
- Route Propagation
- Associations

These work together to control traffic flow.

---

# 8. What is an Attachment?

An attachment connects a resource to the Transit Gateway.

Supported attachment types include:

- VPC
- Site-to-Site VPN
- Direct Connect Gateway
- Transit Gateway Peering

Without an attachment, the resource cannot communicate through the Transit Gateway.

---

# 9. VPC Attachment

Example:

```
VPC

↓

Attachment

↓

Transit Gateway
```

Each participating VPC must have its own attachment.

---

# 10. Transit Gateway Route Table

A Transit Gateway maintains its own route tables.

Important:

These are **different** from VPC route tables.

Traffic is evaluated through:

```
VPC Route Table

↓

Transit Gateway Route Table

↓

Destination Attachment
```

---

# 11. Packet Flow

Example:

```
EC2

↓

VPC Route Table

↓

Transit Gateway

↓

TGW Route Table

↓

Destination VPC

↓

EC2
```

Understanding this flow is essential for troubleshooting.

---

# 12. Route Propagation

Route propagation allows attachment routes to be automatically added to a Transit Gateway route table.

Instead of manually creating routes for every VPC, TGW can learn routes from attached networks.

This reduces administrative overhead.

---

# 13. Route Association

Each attachment must be associated with a Transit Gateway route table.

Think of it as:

```
Attachment

↓

Uses

↓

This Route Table
```

An attachment can be associated with one TGW route table at a time.

---

# 14. Route Propagation vs Association

This is a common interview question.

### Route Association

Defines **which TGW route table an attachment uses**.

### Route Propagation

Defines **which routes an attachment advertises into a TGW route table**.

Remember:

Association = Which table to consult.

Propagation = Which routes are learned.

---

# 15. Basic Architecture

```
                +----------------------+
                |  Transit Gateway     |
                +----------+-----------+
                           |
        -----------------------------------------
        |                 |                     |
+-------+------+  +--------+-------+  +---------+------+
| Production   |  | Development   |  | Shared Services |
| VPC          |  | VPC           |  | VPC             |
+--------------+  +----------------+ +-----------------+
```

Traffic flows through the Transit Gateway instead of direct peering.

---

# 16. TGW vs VPC Peering

| Feature | VPC Peering | Transit Gateway |
|---------|-------------|-----------------|
| Architecture | Mesh | Hub-and-Spoke |
| Transitive Routing | No | Yes (through TGW routing) |
| Scalability | Limited | High |
| Centralized Routing | No | Yes |
| Hybrid Connectivity | No | Yes |
| Easier to Manage | No | Yes |

---

# 17. Typical Production Use Cases

- Enterprise networking
- Shared Services VPC
- Centralized logging
- Centralized monitoring
- Multi-account AWS environments
- Hybrid cloud
- Disaster Recovery

---

# 18. Best Practices

- Allocate non-overlapping CIDRs.
- Use separate TGW route tables for different environments.
- Use tags for attachments.
- Monitor attachment health.
- Keep routing simple and documented.

---

# 19. Common Mistakes

❌ Treating TGW like VPC Peering.

❌ Forgetting to associate an attachment with a TGW route table.

❌ Forgetting route propagation.

❌ Overlapping CIDR ranges.

❌ Ignoring VPC route tables.

---

# 20. Production Scenario

### Problem

A new Development VPC cannot communicate with Production.

Checklist:

- Is the VPC attached to the TGW?
- Is the attachment available?
- Is it associated with the correct TGW route table?
- Are routes propagated?
- Are VPC route tables updated?
- Are Security Groups allowing the traffic?

---

# 21. Interview Questions

## Question 1

What is AWS Transit Gateway?

### Answer

AWS Transit Gateway is a regional network hub that connects multiple VPCs, VPNs, and Direct Connect gateways using a centralized routing architecture.

---

## Question 2

Why was Transit Gateway introduced?

### Answer

To simplify networking by replacing complex full-mesh VPC peering with a scalable hub-and-spoke model.

---

## Question 3

What is a Transit Gateway Attachment?

### Answer

An attachment connects a VPC, VPN, Direct Connect Gateway, or another Transit Gateway to the Transit Gateway.

---

## Question 4

What is the difference between Route Association and Route Propagation?

### Answer

Association determines which TGW route table an attachment uses. Propagation determines which routes an attachment advertises into a TGW route table.

---

## Question 5

When should you choose Transit Gateway over VPC Peering?

### Answer

When connecting many VPCs, supporting hybrid connectivity, implementing centralized routing, or building multi-account enterprise environments.

---

# 22. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Create:

- Production VPC
- Development VPC
- Shared Services VPC

---

## Lab 2

Create a Transit Gateway.

---

## Lab 3

Create VPC attachments for all three VPCs.

---

## Lab 4

Associate each attachment with a TGW route table.

---

## Lab 5

Enable route propagation and verify communication between VPCs.

---

# 23. One-Page Revision

```
Production VPC
        │
Development VPC
        │
Shared Services
        │
Monitoring VPC
        │
        ▼
 +--------------------+
 | Transit Gateway    |
 +--------------------+
```

Remember:

- Hub-and-Spoke architecture
- Centralized routing
- Supports VPC, VPN, Direct Connect
- Uses Attachments
- Uses TGW Route Tables
- Uses Associations
- Uses Route Propagation
- Scales better than VPC Peering

---

# Think Like a Production Engineer

Transit Gateway is not just another networking service—it is the backbone of large AWS environments.

When designing enterprise networks:

1. Plan CIDR ranges carefully.
2. Group environments with separate TGW route tables where appropriate.
3. Understand the complete packet path: VPC Route Table → TGW → TGW Route Table → Destination Attachment.
4. Validate both VPC and TGW routing during troubleshooting.
5. Prefer TGW over a growing mesh of VPC peering connections.

# End of Part 1
