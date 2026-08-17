# Transit Gateway

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 18
>
> Transit Gateway – Part 4 (Production, Troubleshooting & Best Practices)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Troubleshoot Transit Gateway issues
- Design enterprise TGW architectures
- Optimize routing
- Monitor Transit Gateway
- Understand AWS quotas
- Design highly available networking
- Answer advanced interview questions

---

# 70. Enterprise Production Architecture

Consider a multinational company.

```
                           AWS Organization

                                  │

                     Networking Account (Central)

                                  │

                         Transit Gateway (TGW)

          ┌─────────────┬──────────────┬──────────────┐

          │             │              │              │

      Production      Development     Shared      Security

         VPC             VPC          Services       VPC

          │              │               │             │

          └──────────────┴───────────────┴─────────────┘

                         │

              Direct Connect Gateway

                         │

                Corporate Data Center
```

Advantages

- Centralized networking
- Easy management
- Lower operational cost
- Better security
- Easier troubleshooting

---

# 71. Packet Flow in Transit Gateway

One of the most common interview questions.

Suppose:

```
EC2

10.0.1.10

↓

Database

172.16.1.20
```

Packet Flow

```
EC2

↓

Subnet Route Table

↓

Transit Gateway Attachment

↓

Transit Gateway

↓

TGW Route Table

↓

Destination Attachment

↓

Destination VPC

↓

Destination Route Table

↓

Database
```

If communication fails, inspect every step in this sequence.

---

# 72. Troubleshooting Checklist

Whenever traffic fails:

### Step 1

Is the TGW Attachment in **Available** state?

---

### Step 2

Is the attachment associated with the correct TGW Route Table?

---

### Step 3

Has route propagation occurred?

---

### Step 4

Does the VPC Route Table contain the correct TGW route?

---

### Step 5

Do Security Groups allow the traffic?

---

### Step 6

Are Network ACLs blocking traffic?

---

### Step 7

Does the destination application listen on the expected port?

---

### Step 8

Review VPC Flow Logs and CloudWatch metrics.

---

# 73. Monitoring Transit Gateway

Useful monitoring tools:

- Amazon CloudWatch
- VPC Flow Logs
- AWS CloudTrail
- Transit Gateway Route Tables
- Transit Gateway Attachments
- CloudWatch Alarms

Recommended alarms:

- Attachment State Changes
- VPN Tunnel Down
- High Packet Drops
- High Network Errors

---

# 74. High Availability Design

Always avoid a single point of failure.

Recommended architecture:

```
                Direct Connect 1

                        │

                Direct Connect Gateway

                        │

                Transit Gateway

                        │

        ┌───────────────┴───────────────┐

        │                               │

   Availability Zone A          Availability Zone B

        │                               │

      EC2                             EC2
```

Also:

- Use multiple Availability Zones.
- Deploy redundant VPN tunnels.
- Consider multiple Direct Connect connections for mission-critical workloads.

---

# 75. AWS Limits (Check Current AWS Documentation)

Some important quotas include:

- Number of VPC attachments per TGW
- Number of TGW Route Tables
- Number of propagated routes
- Number of static routes

These quotas can change over time, and many are adjustable through AWS Service Quotas.

---

# 76. Cost Considerations

Transit Gateway pricing typically includes:

- Hourly charge per attachment
- Data processing charges

Factors affecting cost:

- Number of attached VPCs
- VPN attachments
- Direct Connect attachments
- Data transferred through the TGW

Design your network to minimize unnecessary east-west traffic.

---

# 77. Best Practices

✔ Use one centralized Transit Gateway for an AWS Organization.

✔ Separate Production and Development using different TGW Route Tables.

✔ Enable route propagation where appropriate.

✔ Use meaningful tags for attachments.

✔ Keep CIDR blocks non-overlapping.

✔ Document every attachment and route.

✔ Monitor VPN tunnels and TGW attachment health.

✔ Regularly review route tables to remove unused entries.

---

# 78. Common Mistakes

❌ Forgetting VPC Route Table updates.

❌ Forgetting TGW Route Table association.

❌ Missing route propagation.

❌ Overlapping CIDR ranges.

❌ Connecting every VPC with VPC Peering instead of using TGW.

❌ Sharing one route table across all environments without isolation.

❌ No monitoring or alerting.

---

# 79. Real Production Scenario 1

### Problem

Production cannot reach Shared Services.

Investigation:

- TGW Attachment ✔
- TGW Route Table ✔
- VPC Route Table ❌ Missing route

Root Cause:

The Production subnet route table did not contain a route pointing to the Transit Gateway.

---

# 80. Real Production Scenario 2

### Problem

On-Premises users cannot access an EC2 instance.

Investigation:

- VPN Tunnel ✔
- TGW Attachment ✔
- Route Propagation ✔
- Security Group ❌ Port 443 blocked

Root Cause:

Traffic reached the VPC, but the EC2 Security Group denied HTTPS.

---

# 81. Real Production Scenario 3

### Problem

Development unexpectedly accessed Production.

Investigation:

Both environments were associated with the same TGW Route Table.

Root Cause:

Network isolation was not implemented.

Solution:

Create separate TGW Route Tables and control propagation.

---

# 82. Real Production Scenario 4

### Problem

A new VPC was attached to the Transit Gateway but communication failed.

Root Cause:

The attachment existed but was not associated with a TGW Route Table.

---

# 83. Decision Matrix

| Requirement | Recommended Solution |
|-------------|----------------------|
| Connect 2 VPCs | VPC Peering |
| Connect many VPCs | Transit Gateway |
| Connect On-Premises | VPN or Direct Connect + TGW |
| Multi-Account Networking | Transit Gateway + AWS RAM |
| Centralized Routing | Transit Gateway |
| Shared Services | Transit Gateway |

---

# 84. Interview Questions

## Question 16

How is Transit Gateway different from a traditional router?

### Answer

Transit Gateway is a managed AWS networking service. AWS operates the infrastructure, while customers configure attachments, route tables, and propagation. Unlike a traditional router, there is no device to patch or maintain.

---

## Question 17

Does Transit Gateway replace VPC Peering?

### Answer

Not always.

For two VPCs, VPC Peering is often simpler. Transit Gateway is preferred when connecting many VPCs, hybrid networks, or multiple AWS accounts.

---

## Question 18

Can a Transit Gateway connect VPCs in different AWS accounts?

### Answer

Yes.

Use AWS Resource Access Manager (RAM) to share the Transit Gateway across AWS accounts.

---

## Question 19

Why are VPC Route Tables still required when using Transit Gateway?

### Answer

Traffic must first reach the Transit Gateway. VPC Route Tables determine whether traffic is forwarded to the TGW attachment.

---

## Question 20

What are the most common reasons Transit Gateway communication fails?

### Answer

- Missing VPC route
- Missing TGW route
- Incorrect TGW Route Table association
- Missing route propagation
- Security Group restrictions
- Network ACL restrictions
- Overlapping CIDR ranges

---

## Question 21

Can a Transit Gateway attach to another Transit Gateway?

### Answer

Yes. AWS supports **Transit Gateway Peering Attachments**, allowing Transit Gateways in different Regions to exchange traffic over the AWS global network.

---

## Question 22

How would you isolate Development from Production using Transit Gateway?

### Answer

Create separate TGW Route Tables, associate the respective VPC attachments with the appropriate route tables, and control which routes are propagated to each table.

---

## Question 23

What happens if route propagation is disabled?

### Answer

The Transit Gateway will not automatically learn routes from that attachment. You must create static routes if communication is still required.

---

## Question 24

Can Transit Gateway be used with Direct Connect?

### Answer

Yes. Direct Connect integrates through a Direct Connect Gateway, which can connect to a Transit Gateway to provide private connectivity to multiple VPCs.

---

## Question 25

What logs and tools would you use to troubleshoot Transit Gateway issues?

### Answer

- VPC Flow Logs
- Amazon CloudWatch
- AWS CloudTrail
- Transit Gateway Route Tables
- Transit Gateway Attachments
- EC2 Reachability testing (where applicable)
- Security Group and Network ACL review

---

# 85. Hands-on Labs

## Lab 16

Create:

- Production VPC
- Development VPC
- Shared Services VPC

Connect all through a Transit Gateway.

---

## Lab 17

Create separate TGW Route Tables for Production and Development.

Verify that Development cannot access Production.

---

## Lab 18

Share the Transit Gateway with another AWS account using AWS RAM.

Create an attachment from the second account.

---

## Lab 19

Configure a simulated hybrid environment:

- Transit Gateway
- Site-to-Site VPN
- Production VPC

Verify route propagation.

---

## Lab 20

Troubleshooting Exercise:

Break connectivity by:

- Removing a VPC route
- Disabling route propagation
- Blocking traffic with a Security Group

Identify and resolve each issue.

---

# 86. One-Page Revision

```
Small Environment

VPC Peering

↓

Large Environment

Transit Gateway

↓

Hybrid Cloud

Transit Gateway + VPN

↓

Enterprise

Transit Gateway + AWS RAM + Direct Connect
```

Remember:

- Hub-and-Spoke architecture
- Centralized routing
- VPC Attachments
- TGW Route Tables
- Route Association
- Route Propagation
- AWS RAM
- Cross-Account support
- Hybrid networking
- High Availability
- Monitoring
- Troubleshooting

---

# Think Like a Production Engineer

A Transit Gateway is the **network backbone** of many enterprise AWS environments.

When designing a network:

1. Keep CIDR blocks unique.
2. Centralize routing with a Transit Gateway.
3. Isolate environments using separate TGW Route Tables.
4. Validate both VPC and TGW routing during troubleshooting.
5. Monitor attachments, VPN tunnels, and routing health.
6. Prefer simplicity—avoid unnecessary route complexity.

Design your TGW as if your organization will double in size. A scalable architecture today prevents painful redesigns tomorrow.

# End of Chapter 18 – Transit Gateway
