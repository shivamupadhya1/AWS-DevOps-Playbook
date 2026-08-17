# Transit Gateway

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 18
>
> Transit Gateway – Part 2

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Design multi-account Transit Gateway architectures
- Understand AWS Resource Access Manager (RAM)
- Configure multiple TGW Route Tables
- Understand Route Isolation
- Understand Route Propagation
- Configure Cross-Region Transit Gateway Peering
- Implement Transit Gateway using AWS CLI and Terraform

---

# 24. Multi-Account Architecture

Large organizations rarely have a single AWS account.

Example:

```
AWS Organization

├── Security Account

├── Shared Services Account

├── Production Account

├── Development Account

└── Testing Account
```

Instead of creating a Transit Gateway in every account, organizations usually deploy **one central Transit Gateway** in a networking or shared services account and share it with other AWS accounts.

---

# 25. AWS Resource Access Manager (RAM)

AWS RAM allows you to share supported AWS resources across accounts.

For Transit Gateway, RAM enables one account to own the TGW while other accounts create attachments.

```
Networking Account

↓

Transit Gateway

↓

AWS RAM Share

↓

Production Account

↓

Development Account

↓

Testing Account
```

This avoids deploying separate TGWs in every account.

---

# 26. Benefits of AWS RAM

- Centralized networking
- Lower operational overhead
- Consistent routing
- Easier governance
- Reduced costs
- Better scalability

---

# 27. Enterprise Architecture

```
                 AWS Organization

                        │

          +------------------------------+

          |     Networking Account       |

          |      Transit Gateway         |

          +--------------+---------------+

                         |

      --------------------------------------------

      |                 |                |

Production        Development      Shared Services

Account           Account          Account
```

All traffic passes through the centralized Transit Gateway.

---

# 28. Multiple TGW Route Tables

A Transit Gateway can have multiple route tables.

Example:

```
Production Route Table

Development Route Table

Shared Services Route Table
```

This allows different traffic policies for different environments.

---

# 29. Why Multiple Route Tables?

Imagine:

```
Production

↓

Should communicate only with

↓

Shared Services
```

Development should not access Production.

Separate TGW route tables enforce this network isolation.

---

# 30. Route Isolation

Example:

```
Production Route Table

Production

↓

Shared Services

✓ Allowed

↓

Development

✗ Not Allowed
```

This is a common enterprise security pattern.

---

# 31. Route Propagation

Attachments can automatically advertise their routes into TGW route tables.

Example:

```
Production VPC

↓

Attachment

↓

Advertise

↓

10.0.0.0/16
```

The TGW learns the route automatically.

---

# 32. Static Routes

You can also create routes manually.

Use static routes when:

- Integrating third-party appliances
- Testing
- Creating blackhole routes
- Custom routing requirements

---

# 33. Blackhole Routes

A Blackhole Route intentionally drops traffic.

Example:

```
Destination

10.50.0.0/16

↓

Blackhole
```

Useful for preventing communication to specific networks.

---

# 34. Cross-Region Transit Gateway Peering

Transit Gateways in different AWS Regions can be peered.

```
Mumbai TGW

↓

TGW Peering

↓

Singapore TGW
```

This enables private connectivity between regional networks.

---

# 35. Cross-Region Architecture

```
Mumbai Region

Production VPC

↓

Mumbai TGW

↓

TGW Peering

↓

Singapore TGW

↓

Singapore Production VPC
```

Traffic uses the AWS global backbone.

---

# 36. Packet Flow

```
EC2

↓

VPC Route Table

↓

TGW Attachment

↓

Transit Gateway

↓

TGW Route Table

↓

Destination Attachment

↓

Destination VPC

↓

EC2
```

Understanding this sequence is critical for troubleshooting.

---

# 37. AWS CLI - Create TGW

```bash
aws ec2 create-transit-gateway \
  --description "Enterprise Transit Gateway"
```

---

# 38. AWS CLI - Describe TGW

```bash
aws ec2 describe-transit-gateways
```

Useful for checking:

- State
- ASN
- Owner
- Attachments

---

# 39. Terraform Example

```hcl
resource "aws_ec2_transit_gateway" "main" {

  description = "Enterprise TGW"

  tags = {
    Name = "Enterprise-TGW"
  }

}
```

---

# 40. Terraform Attachment Example

```hcl
resource "aws_ec2_transit_gateway_vpc_attachment" "prod" {

  subnet_ids = [
    aws_subnet.private1.id,
    aws_subnet.private2.id
  ]

  transit_gateway_id = aws_ec2_transit_gateway.main.id

  vpc_id = aws_vpc.production.id

}
```

---

# 41. Production Scenario

### Problem

Development VPC cannot reach Shared Services.

Checklist:

- TGW attachment exists
- Attachment is Available
- Associated with correct TGW route table
- Route propagated
- VPC route table updated
- Security Groups allow traffic

---

# 42. Best Practices

- Use a dedicated Networking Account.
- Share TGW through AWS RAM.
- Separate Production and Development route tables.
- Use descriptive attachment names.
- Monitor attachment health.
- Enable VPC Flow Logs for troubleshooting.

---

# 43. Common Mistakes

❌ Using one TGW route table for every environment.

❌ Forgetting route propagation.

❌ Ignoring VPC route tables.

❌ Assuming TGW automatically bypasses Security Groups.

❌ Forgetting to accept RAM shares in another account.

---

# 44. Interview Questions

## Question 6

What is AWS RAM?

### Answer

AWS Resource Access Manager (RAM) allows supported AWS resources, including Transit Gateways, to be securely shared across multiple AWS accounts.

---

## Question 7

Why use multiple TGW route tables?

### Answer

To isolate traffic between environments such as Production, Development, and Shared Services while maintaining centralized routing.

---

## Question 8

Can Transit Gateway connect multiple AWS accounts?

### Answer

Yes. A Transit Gateway can be shared across accounts using AWS RAM, allowing centralized networking within an AWS Organization.

---

## Question 9

What is a Blackhole Route?

### Answer

A Blackhole Route intentionally drops traffic destined for a specific network, preventing communication.

---

## Question 10

Does Transit Gateway support Cross-Region connectivity?

### Answer

Yes. Transit Gateways in different AWS Regions can be connected using Transit Gateway Peering.

---

# 45. Hands-on Labs

## Lab 6

Create two TGW route tables:

- Production
- Development

Associate different VPC attachments with each.

---

## Lab 7

Share a Transit Gateway using AWS RAM.

---

## Lab 8

Create a TGW VPC attachment using Terraform.

---

## Lab 9

Create a Blackhole Route and verify that traffic is dropped.

---

## Lab 10

Inspect packet flow using VPC Flow Logs.

---

# 46. One-Page Revision

```
VPC

↓

Attachment

↓

Transit Gateway

↓

TGW Route Table

↓

Destination Attachment

↓

Destination VPC
```

Remember:

- AWS RAM enables cross-account sharing.
- TGW supports multiple route tables.
- Route Association selects the route table.
- Route Propagation advertises routes.
- Blackhole routes intentionally discard traffic.
- Cross-Region TGW Peering connects regional networks.

---

# Think Like a Production Engineer

In enterprise environments:

- Deploy one centralized Transit Gateway in a networking account.
- Share it with other AWS accounts using AWS RAM.
- Use separate TGW route tables to isolate Production, Development, and Shared Services.
- Always troubleshoot both VPC route tables and TGW route tables.
- Keep routing simple, documented, and scalable.

# End of Part 2
