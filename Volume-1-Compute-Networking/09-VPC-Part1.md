# Amazon Virtual Private Cloud (VPC)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 09
>
> Part 1 – VPC Fundamentals & Architecture

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand what a VPC is
- Design a production-ready VPC
- Understand CIDR blocks
- Understand Subnets
- Understand Availability Zones
- Differentiate Public vs Private Subnets
- Understand AWS networking fundamentals
- Answer VPC interview questions confidently

---

# 1. What is a VPC?

Amazon Virtual Private Cloud (VPC) is a logically isolated virtual network inside AWS where you launch and manage your cloud resources.

Think of a VPC as your **own private data center inside AWS**.

It provides complete control over:

- IP address range
- Subnets
- Routing
- Internet connectivity
- Security
- Network isolation

Without a VPC, AWS resources cannot communicate over a network.

---

# 2. Why Do We Need a VPC?

Imagine a company moving from an on-premises data center to AWS.

In a physical data center, they have:

- Private IP ranges
- Routers
- Switches
- Firewalls
- Servers

AWS provides the same networking capabilities using VPC.

Instead of purchasing hardware, AWS lets you create software-defined networks.

---

# 3. Real-Life Analogy

Imagine a large apartment complex.

- AWS Region → City
- Availability Zone → Building
- VPC → Apartment Complex
- Subnet → Floor
- EC2 Instance → Apartment
- Security Group → Apartment Door Lock
- NACL → Security Guard at Building Entrance

Each apartment is isolated, but residents can communicate based on the building's rules.

---

# 4. VPC Architecture

```
                 AWS Region
                     │
      ┌─────────────────────────┐
      │           VPC           │
      │  10.0.0.0/16            │
      │                         │
      │  ┌──────────────┐       │
      │  │ PublicSubnet │       │
      │  │10.0.1.0/24   │       │
      │  └──────────────┘       │
      │                         │
      │  ┌──────────────┐       │
      │  │PrivateSubnet │       │
      │  │10.0.2.0/24   │       │
      │  └──────────────┘       │
      └─────────────────────────┘
```

---

# 5. VPC Components

A production VPC consists of:

- CIDR Block
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs
- Elastic IP
- VPC Endpoints
- Peering
- Transit Gateway

These will be covered in later chapters.

---

# 6. AWS Regions and Availability Zones

A VPC belongs to **one AWS Region**.

Example:

```
ap-south-1 (Mumbai)
```

Inside a Region are multiple Availability Zones.

Example:

```
ap-south-1a

ap-south-1b

ap-south-1c
```

Subnets are always created inside a single Availability Zone.

A subnet **cannot span multiple Availability Zones**.

---

# 7. CIDR Block

Every VPC requires a CIDR block.

Example:

```
10.0.0.0/16
```

CIDR defines the IP address range available within the VPC.

Examples:

```
10.0.0.0/16

172.16.0.0/16

192.168.0.0/16
```

These are private IPv4 ranges defined by RFC 1918.

---

# 8. CIDR Notation

Example:

```
10.0.0.0/16
```

- `10.0.0.0` → Network address
- `/16` → Prefix length (first 16 bits are fixed)

A `/16` network provides:

```
65,536 total IP addresses
```

Not all are usable because AWS reserves some addresses in every subnet.

---

# 9. Common CIDR Sizes

| CIDR | Total IPs |
|------|----------:|
| /16 | 65,536 |
| /17 | 32,768 |
| /18 | 16,384 |
| /19 | 8,192 |
| /20 | 4,096 |
| /21 | 2,048 |
| /22 | 1,024 |
| /23 | 512 |
| /24 | 256 |
| /25 | 128 |
| /26 | 64 |
| /27 | 32 |
| /28 | 16 |

Interviewers often ask you to estimate the approximate size rather than calculate every address.

---

# 10. AWS Reserved IP Addresses

Suppose your subnet is:

```
10.0.1.0/24
```

AWS reserves the first four IP addresses and the last IP address.

Example:

```
10.0.1.0
Network Address

10.0.1.1
VPC Router

10.0.1.2
Amazon DNS

10.0.1.3
Reserved

10.0.1.255
Broadcast (Reserved by AWS)
```

So, a `/24` subnet has:

- Total IPs: 256
- Usable IPs: 251

---

# 11. What is a Subnet?

A subnet is a smaller network created from the VPC CIDR.

Example:

VPC

```
10.0.0.0/16
```

Subnets

```
10.0.1.0/24

10.0.2.0/24

10.0.3.0/24

10.0.4.0/24
```

Each subnet contains its own range of IP addresses.

---

# 12. Why Create Multiple Subnets?

Reasons include:

- High Availability
- Better Security
- Isolation
- Scalability
- Fault Tolerance

Instead of placing everything in one subnet, distribute resources across multiple subnets and Availability Zones.

---

# 13. Public Subnet

A subnet is considered **public** when its route table contains a route to an Internet Gateway.

Typical resources:

- Application Load Balancer
- Bastion Host
- NAT Gateway
- Public EC2

---

# 14. Private Subnet

A private subnet does not have a direct route to the Internet Gateway.

Typical resources:

- Application Servers
- Databases
- Internal Services
- EKS Worker Nodes (common production pattern)

---

# 15. Public vs Private Subnet

| Public Subnet | Private Subnet |
|--------------|----------------|
| Internet access | No direct internet access |
| Hosts ALB | Hosts databases |
| Bastion Host | Backend applications |
| Public EC2 | Internal services |
| More exposed | More secure |

---

# 16. Multi-AZ Architecture

A highly available design distributes workloads across Availability Zones.

```
                 VPC

        ┌───────────────────┐

      AZ-A              AZ-B

Public             Public

Private            Private

ALB                ALB

App                App

Database           Database
```

If one Availability Zone becomes unavailable, the application can continue running in another.

---

# 17. High Availability

Never deploy all servers in a single Availability Zone.

Always distribute:

- Load Balancers
- Application Servers
- Databases
- Kubernetes Worker Nodes

This reduces the impact of an AZ failure.

---

# 18. Best Practices

- Use one VPC per environment when appropriate (Dev/Test/Prod).
- Plan your CIDR ranges before creating resources.
- Spread workloads across multiple Availability Zones.
- Keep databases in private subnets.
- Avoid exposing backend services directly to the internet.
- Use meaningful names and tags for VPC resources.

---

# 19. Common Mistakes

❌ Creating a VPC with an IP range that overlaps with on-premises networks.

❌ Deploying everything in one subnet.

❌ Putting databases in public subnets.

❌ Running production applications in a single Availability Zone.

❌ Using overly small CIDR blocks that limit future growth.

---

# 20. Interview Questions

## Question 1

What is a VPC?

### Perfect Answer

A VPC is a logically isolated virtual network in AWS where we launch cloud resources. It allows us to define IP ranges, create subnets, configure routing, and control network security.

---

## Question 2

Why do we need a VPC?

### Perfect Answer

A VPC provides network isolation and control. It allows organizations to design secure and scalable cloud networks similar to an on-premises data center.

---

## Question 3

Can a subnet span multiple Availability Zones?

### Perfect Answer

No. A subnet belongs to a single Availability Zone. To achieve high availability, we create multiple subnets across multiple Availability Zones.

---

## Question 4

What makes a subnet public?

### Perfect Answer

A subnet becomes public when its route table contains a route to an Internet Gateway, allowing resources with public IP addresses to communicate with the internet.

---

## Question 5

Why are databases usually placed in private subnets?

### Perfect Answer

Databases should not be directly accessible from the internet. Placing them in private subnets reduces the attack surface while allowing application servers to connect internally.

---

# 21. Amazon Follow-up Questions

### Question

Can resources in different subnets communicate?

### Perfect Answer

Yes. As long as they are within the same VPC and security rules allow the traffic, resources in different subnets can communicate using the VPC's local routing.

---

### Question

Can two VPCs communicate by default?

### Perfect Answer

No. Separate VPCs are isolated by default. Communication requires mechanisms such as VPC Peering, Transit Gateway, or VPN connections.

---

### Question

What happens if you choose a very small CIDR block?

### Perfect Answer

You may run out of available IP addresses as your infrastructure grows, making future expansion difficult without redesigning the network.

---

# 22. Hands-on Labs (To Perform Later)

## Lab 1

Create a VPC with CIDR `10.0.0.0/16`.

---

## Lab 2

Create two public subnets in different Availability Zones.

---

## Lab 3

Create two private subnets in different Availability Zones.

---

## Lab 4

Launch EC2 instances in each subnet and observe the assigned private IP addresses.

---

## Lab 5

Experiment with different subnet CIDR sizes and calculate available IP addresses.

---

# 23. One-Page Revision

```
AWS Region
     │
    VPC
     │
 ┌──────────────┐
 │              │
Public      Private
Subnet      Subnet
 │              │
ALB          App
Bastion      DB
```

Remember:

- VPC = Isolated network
- CIDR = IP range
- Subnet = Portion of VPC
- Public Subnet = Route to Internet Gateway
- Private Subnet = No direct route to Internet Gateway
- One subnet belongs to one Availability Zone
- Use Multi-AZ for High Availability

---

# Think Like a Production Engineer

Don't ask:

> "How do I create a VPC?"

Ask:

> "Can this network scale, remain secure, and survive an Availability Zone failure?"

That mindset distinguishes a production-ready design from a basic deployment.

# End of Part 1
