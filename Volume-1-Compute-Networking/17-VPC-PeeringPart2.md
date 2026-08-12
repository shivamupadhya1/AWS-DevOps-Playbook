# VPC Peering

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 17
>
> VPC Peering – Part 2

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Non-Transitive Routing
- Configure Cross-Region VPC Peering
- Configure Cross-Account VPC Peering
- Understand DNS Resolution
- Create Peering using AWS CLI
- Create Peering using Terraform
- Troubleshoot VPC Peering
- Design Production Architectures

---

# 24. Non-Transitive Routing

This is the **most frequently asked interview question** on VPC Peering.

Suppose we have three VPCs.

```
VPC-A
10.0.0.0/16

        │
        │
     Peering
        │

VPC-B
172.16.0.0/16

        │
        │
     Peering
        │

VPC-C
192.168.0.0/16
```

Question:

Can VPC-A communicate with VPC-C?

Answer:

```
NO
```

---

# 25. Why?

AWS does NOT allow one VPC to act as a router.

Even though:

```
A ↔ B

B ↔ C
```

AWS will NOT allow

```
A ↔ C
```

through B.

This is called

```
Non-Transitive Routing
```

---

# 26. Example

```
Application

↓

VPC-A

↓

Database

VPC-C
```

If traffic must pass through

```
VPC-B
```

AWS drops it.

---

# 27. Why AWS Designed It This Way

Reasons:

- Security
- Predictable routing
- Simpler route management
- Better isolation
- Reduced accidental exposure

For many interconnected VPCs, AWS recommends using **Transit Gateway** instead.

---

# 28. Cross-Region VPC Peering

AWS also supports peering between VPCs in different regions.

Example:

```
Mumbai

↓

Peering

↓

Singapore
```

Traffic still uses the AWS global network.

---

# 29. Cross-Region Architecture

```
+-------------------+
| Mumbai Region     |
| VPC-A             |
|10.0.0.0/16        |
+---------+---------+
          │
          │
     Cross-Region
        Peering
          │
          │
+---------+---------+
| Singapore Region  |
| VPC-B             |
|172.16.0.0/16      |
+-------------------+
```

---

# 30. Use Cases

- Disaster Recovery
- Multi-Region Applications
- Centralized Logging
- Global Monitoring
- Shared Authentication Services

---

# 31. Cross-Account VPC Peering

Different AWS accounts can also establish a peering connection.

Example

```
AWS Account A

↓

VPC-A

↓

Peering

↓

VPC-B

↓

AWS Account B
```

---

# 32. Cross-Account Workflow

```
Account A

Create Peering Request

↓

Account B

Accept Request

↓

Update Route Tables

↓

Update Security Groups

↓

Communication Established
```

---

# 33. DNS Resolution

By default, instances communicate using private IP addresses.

AWS also allows private DNS resolution across a peering connection for supported scenarios.

This enables applications to continue using DNS names instead of hardcoded IP addresses.

---

# 34. Peering Lifecycle

```
Pending Acceptance

↓

Active

↓

Deleted

or

↓

Rejected

or

↓

Expired
```

Always verify that the connection state is **Active** before troubleshooting routes or security rules.

---

# 35. AWS CLI - Create Peering

```bash
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-11111111 \
  --peer-vpc-id vpc-22222222
```

---

# 36. AWS CLI - Accept Peering

```bash
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-1234567890abcdef
```

---

# 37. AWS CLI - Describe Peering

```bash
aws ec2 describe-vpc-peering-connections
```

Useful for checking:

- Status
- VPC IDs
- Owner IDs
- Region
- CIDR information

---

# 38. Terraform Example

```hcl
resource "aws_vpc_peering_connection" "main" {

  vpc_id      = aws_vpc.app.id
  peer_vpc_id = aws_vpc.db.id
  auto_accept = true

  tags = {
    Name = "App-DB-Peering"
  }

}
```

---

# 39. Terraform Route Example

```hcl
resource "aws_route" "app_to_db" {

  route_table_id            = aws_route_table.app.id

  destination_cidr_block    = "172.16.0.0/16"

  vpc_peering_connection_id = aws_vpc_peering_connection.main.id

}
```

Repeat similar configuration for the route table in the peer VPC.

---

# 40. Security Group Example

Application Security Group

```
Outbound

↓

Database CIDR

172.16.0.0/16
```

Database Security Group

```
Inbound

↓

3306

↓

Source

10.0.0.0/16
```

---

# 41. Route Table Example

VPC-A

| Destination | Target |
|-------------|--------|
|172.16.0.0/16|Peering|

---

VPC-B

| Destination | Target |
|-------------|--------|
|10.0.0.0/16|Peering|

Both directions are required.

---

# 42. Production Architecture

```
                    Shared Services VPC
                     172.16.0.0/16
                           │
                    VPC Peering
                           │
                 ---------------------
                 │                   │
          Production VPC      Development VPC
           10.0.0.0/16         192.168.0.0/16
```

Shared Services may contain:

- DNS
- Active Directory
- Monitoring
- Logging
- Artifact Repository

Remember:

Production cannot automatically reach Development through Shared Services because peering is **not transitive**.

---

# 43. When Should You Use VPC Peering?

Good choices:

- Two VPCs
- Simple architecture
- Low management overhead
- Small environments
- Stable connectivity requirements

---

# 44. When Should You NOT Use VPC Peering?

Avoid VPC Peering when:

- Many VPCs require full mesh connectivity.
- Frequent new VPCs are created.
- Centralized routing is needed.
- Large multi-account environments exist.

In these cases, Transit Gateway is usually a better solution.

---

# 45. Best Practices

- Design CIDR ranges before creating VPCs.
- Leave room for future network expansion.
- Keep route tables simple.
- Use descriptive tags.
- Enable VPC Flow Logs for troubleshooting.
- Restrict Security Group rules to required CIDRs.

---

# 46. Common Mistakes

❌ Assuming routes are created automatically.

❌ Forgetting to update both route tables.

❌ Using overlapping CIDRs.

❌ Assuming peering supports transitive routing.

❌ Forgetting Security Group updates.

❌ Ignoring Network ACL restrictions.

---

# 47. Production Scenario 1

### Problem

Application cannot connect to the database after peering.

Checklist:

- Peering status is Active.
- Route exists in VPC-A.
- Route exists in VPC-B.
- Security Groups allow traffic.
- NACLs allow traffic.
- Application is using the correct private IP or DNS.

---

# 48. Production Scenario 2

### Problem

Cross-Account Peering established but communication fails.

Possible causes:

- Missing route table entry.
- Missing Security Group rule.
- Incorrect CIDR.
- Peering request not accepted.

---

# 49. Production Scenario 3

### Problem

Ping works but database connection fails.

Reason:

ICMP is allowed, but database port (for example, 3306) is blocked by the Security Group or NACL.

---

# 50. Production Scenario 4

### Problem

A new VPC is added and teams expect all VPCs to communicate automatically.

Explanation:

VPC Peering does not support transitive routing.

Recommend Transit Gateway for hub-and-spoke connectivity.

---

# 51. Interview Questions

## Question 6

What is Non-Transitive Routing?

### Answer

A VPC Peering connection allows communication only between the two directly connected VPCs. Traffic cannot pass through one peered VPC to reach another.

---

## Question 7

Does VPC Peering support Cross-Region communication?

### Answer

Yes.

AWS supports Cross-Region VPC Peering using the AWS global network.

---

## Question 8

Can two AWS accounts establish VPC Peering?

### Answer

Yes.

One account creates the peering request, and the other account accepts it. Route tables and security rules must then be updated.

---

## Question 9

Can VPC Peering connect overlapping CIDRs?

### Answer

No.

Overlapping CIDR ranges create routing ambiguity and are not supported.

---

## Question 10

When would you choose Transit Gateway instead of VPC Peering?

### Answer

When connecting many VPCs, implementing hub-and-spoke networking, simplifying route management, or building large multi-account environments.

---

# 52. Hands-on Labs (When AWS Account is Ready)

## Lab 6

Create two VPCs in different AWS accounts.

Establish a Cross-Account VPC Peering connection.

---

## Lab 7

Create a Cross-Region VPC Peering connection.

Verify connectivity over private IP addresses.

---

## Lab 8

Create three VPCs:

- VPC-A
- VPC-B
- VPC-C

Peer A↔B and B↔C.

Attempt communication from A to C and observe that it fails because peering is not transitive.

---

## Lab 9

Create the peering connection using the AWS CLI.

---

## Lab 10

Create the peering connection using Terraform.

---

# 53. One-Page Revision

```
VPC-A
10.0.0.0/16
      │
      ▼
 VPC Peering
      ▲
      │
VPC-B
172.16.0.0/16
```

Remember:

- Private communication
- AWS backbone
- Non-overlapping CIDRs
- Manual route updates
- Manual Security Group updates
- No transitive routing
- Supports Cross-Account
- Supports Cross-Region
- Transit Gateway is preferred for many VPCs

---

# Think Like a Production Engineer

When troubleshooting VPC Peering:

1. Verify the connection state is **Active**.
2. Confirm CIDR ranges do not overlap.
3. Check both route tables.
4. Verify Security Group rules.
5. Review Network ACLs if configured.
6. Confirm the application uses the correct private IP or DNS.
7. If multiple VPCs are involved, remember that VPC Peering is **not transitive**.

If the environment is growing beyond a handful of VPCs, evaluate **Transit Gateway** instead of creating numerous peering connections.

# End of Part 2
