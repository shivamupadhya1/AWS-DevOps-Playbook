# Amazon ECS (Elastic Container Service)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 07
>
> Part 2 – Launch Types & Capacity Providers

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand ECS Launch Types
- Explain EC2 vs Fargate
- Understand Capacity Providers
- Explain Fargate Spot
- Optimize ECS costs
- Choose the correct launch type
- Answer production interview questions confidently

---

# 1. ECS Launch Types

Before ECS runs a container, it needs compute resources.

AWS provides two launch types.

```
EC2

or

Fargate
```

Question:

> Where should the container run?

That decision is made by selecting a launch type.

---

# 2. EC2 Launch Type

⭐⭐⭐⭐⭐

Architecture

```
Application

↓

Task Definition

↓

ECS

↓

EC2 Instance

↓

Docker

↓

Container
```

You own the EC2 instances.

AWS manages ECS.

You manage:

- EC2
- AMI
- Patching
- Scaling
- Docker Runtime
- ECS Agent

---

# 3. Advantages of EC2 Launch Type

- Lowest cost for predictable workloads
- Supports Reserved Instances
- Supports Compute Savings Plans
- Full OS access
- SSH access
- Custom AMIs
- GPU support
- Daemon services
- Custom networking

---

# 4. Disadvantages of EC2

You are responsible for:

- OS updates
- Security patches
- Disk management
- Capacity planning
- Instance failures
- ECS Agent health

---

# 5. Production Example

Suppose

```
500 Containers

24×7

Same Load

Every Day
```

Choosing EC2 is usually more cost-effective because long-running predictable workloads benefit from Reserved Instances or Savings Plans.

---

# 6. Fargate Launch Type

⭐⭐⭐⭐⭐

Architecture

```
Application

↓

Task Definition

↓

Amazon ECS

↓

AWS Fargate

↓

Container
```

There is **no EC2 instance** for you to manage.

AWS manages:

- Infrastructure
- Patching
- Scaling of underlying hosts
- Container runtime
- Capacity allocation

You only manage the application.

---

# 7. Advantages of Fargate

- No server management
- No patching
- No AMI maintenance
- Better operational simplicity
- Faster deployments
- Strong workload isolation
- Pay for allocated CPU and memory

---

# 8. Disadvantages of Fargate

- More expensive for always-on workloads
- Less operating system control
- No SSH access to the host
- Limited customization compared to EC2

---

# 9. EC2 vs Fargate

| Feature | EC2 | Fargate |
|----------|------|----------|
| Server Management | Yes | No |
| SSH Access | Yes | No |
| Patching | Customer | AWS |
| Cost for 24×7 Workloads | Lower | Higher |
| Startup Simplicity | Medium | High |
| OS Customization | Full | Limited |
| GPU Support | Yes | Limited by service support |
| Best For | Predictable workloads | Variable workloads |

---

# 10. When Should You Choose EC2?

Choose EC2 when:

- Applications run continuously
- Workload is predictable
- Cost optimization is critical
- You need OS-level customization
- You require specialized instance types
- GPU workloads are involved

---

# 11. When Should You Choose Fargate?

Choose Fargate when:

- Teams don't want to manage servers
- Workload changes frequently
- New microservices are deployed often
- Small DevOps teams
- Fast development cycles
- Batch jobs
- Event-driven applications

---

# 12. Cost Optimization

Suppose

```
200 Tasks

Running

365 Days
```

Fargate becomes expensive.

Better choice:

EC2 + Savings Plan or Reserved Instances.

Suppose

```
20 Tasks

Weekend Only
```

Fargate is often the better operational choice because you only pay while tasks are running.

---

# 13. Capacity Providers

⭐⭐⭐⭐⭐

Capacity Providers tell ECS **where Tasks should run**.

They automate infrastructure selection.

Example

```
Cluster

↓

Capacity Provider

↓

EC2

or

Fargate

or

Fargate Spot
```

---

# 14. Why Capacity Providers?

Before Capacity Providers

You manually selected launch type.

Now ECS can automatically choose based on configuration.

Benefits

- Automatic scaling
- Mixed compute environments
- Better cost optimization
- Simpler deployments

---

# 15. Types of Capacity Providers

```
EC2 Auto Scaling Group

Fargate

Fargate Spot
```

---

# 16. Fargate Spot

⭐⭐⭐⭐⭐

Uses spare AWS capacity.

Benefits

- Significant cost savings
- Great for interruptible workloads

Examples

- CI/CD jobs
- Batch processing
- Data processing
- Development environments

---

# 17. Spot Interruption

Fargate Spot tasks may be interrupted.

AWS provides a notice before stopping the task.

Applications should:

- Handle shutdown gracefully
- Save progress if needed
- Be designed for interruption

---

# 18. Capacity Provider Strategy

Example

```
70%

EC2

30%

Fargate Spot
```

or

```
Base = 2 Tasks on Fargate

Remaining Tasks on Fargate Spot
```

This balances availability and cost.

---

# 19. Savings Plans vs Reserved Instances

Reserved Instances

- Specific instance family
- Fixed commitment
- Good for stable EC2 usage

Compute Savings Plans

- More flexible
- Apply across eligible EC2 usage
- Also benefit Fargate and Lambda compute spend (subject to AWS pricing rules)
- Recommended for many modern workloads

---

# 20. Production Architecture

```
ALB

↓

ECS Service

↓

Capacity Provider

↓

EC2 Auto Scaling Group

↓

Containers
```

OR

```
ALB

↓

ECS Service

↓

Fargate

↓

Containers
```

---

# 21. Production Scenario 1

Problem

Business runs 500 Tasks continuously.

Recommended Solution

EC2 Launch Type with Savings Plans or Reserved Instances.

Reason

Predictable usage lowers long-term cost.

---

# 22. Production Scenario 2

Problem

Startup deploys dozens of short-lived microservices.

Recommended Solution

Fargate.

Reason

No infrastructure management and rapid deployments.

---

# 23. Production Scenario 3

Problem

Nightly ETL jobs.

Recommended Solution

Fargate Spot.

Reason

Lower cost and interruption tolerance.

---

# 24. Production Scenario 4

Problem

Cluster has no remaining CPU.

Check

- ECS Cluster Capacity
- Auto Scaling Group
- Capacity Provider
- Available CPU
- Available Memory

---

# 25. Best Practices

- Use EC2 for predictable, long-running workloads.
- Use Fargate for operational simplicity.
- Use Fargate Spot for interruptible workloads.
- Use Capacity Providers instead of hardcoded launch types where possible.
- Optimize costs with Savings Plans.
- Monitor utilization before scaling.

---

# 26. Common Mistakes

❌ Choosing Fargate for large always-on workloads without cost analysis.

---

❌ Running critical production services entirely on Spot capacity.

---

❌ Not enabling Auto Scaling for EC2 capacity.

---

❌ Mixing launch types without a clear Capacity Provider strategy.

---

# 27. Interview Questions

## Question 1

When would you choose EC2 Launch Type?

### Perfect Answer

I choose EC2 Launch Type for long-running, predictable workloads where cost optimization is important. It allows me to use Reserved Instances or Savings Plans, provides operating system access, and supports greater customization.

---

## Question 2

When would you choose Fargate?

### Perfect Answer

I choose Fargate when I want to avoid infrastructure management. AWS manages the underlying servers, allowing the team to focus only on the application. It is well suited for microservices, variable workloads, and smaller operations teams.

---

## Question 3

What is the biggest difference between EC2 and Fargate?

### Perfect Answer

With EC2 Launch Type, I manage the compute infrastructure, including patching and scaling. With Fargate, AWS manages the infrastructure, and I only define the CPU and memory requirements for my Tasks.

---

## Question 4

What are Capacity Providers?

### Perfect Answer

Capacity Providers define where ECS should run Tasks. They integrate with EC2 Auto Scaling Groups or Fargate and allow ECS to automatically choose and scale the required compute capacity.

---

## Question 5

What is Fargate Spot?

### Perfect Answer

Fargate Spot uses spare AWS capacity to run Tasks at a lower cost. Because Tasks can be interrupted, it is best suited for fault-tolerant or interruptible workloads such as batch jobs and CI/CD pipelines.

---

## Question 6

Your application runs 24×7 for years. Which launch type would you choose?

### Perfect Answer

I would choose EC2 Launch Type because the workload is predictable. This allows the organization to reduce compute costs using Reserved Instances or Savings Plans while maintaining full control over the infrastructure.

---

## Question 7

Your startup launches hundreds of temporary environments every day. Which launch type would you choose?

### Perfect Answer

I would choose Fargate because it removes server management overhead, scales quickly, and charges only for the allocated compute resources while Tasks are running.

---

## Question 8

Can a single ECS Cluster use both EC2 and Fargate?

### Perfect Answer

Yes.

A single ECS Cluster can support both EC2 and Fargate capacity. Capacity Providers determine where individual Services or Tasks are placed.

---

## Question 9

Can Fargate Tasks be interrupted?

### Perfect Answer

Standard Fargate Tasks are not interrupted like Spot capacity. However, Fargate Spot Tasks can be interrupted when AWS reclaims spare capacity.

---

## Question 10

Which option reduces operational overhead the most?

### Perfect Answer

Fargate, because AWS manages the infrastructure, operating system, and underlying compute resources.

---

# 28. Amazon Cross Questions

### Question

If cost is the highest priority and workload is predictable, what would you recommend?

### Perfect Answer

EC2 Launch Type with Savings Plans or Reserved Instances because predictable workloads benefit from lower long-term compute costs.

---

### Question

If the business wants zero server management, what would you recommend?

### Perfect Answer

AWS Fargate.

It allows the team to focus entirely on containers and applications without managing EC2 instances.

---

### Question

Can Capacity Providers improve cost optimization?

### Perfect Answer

Yes.

They allow workloads to be distributed across EC2, Fargate, and Fargate Spot based on business requirements, balancing availability and cost.

---

### Question

Would you deploy a payment service only on Fargate Spot?

### Perfect Answer

No.

Critical services should run on reliable capacity such as EC2 or standard Fargate. Spot capacity is better suited for interruptible workloads.

---

# 29. Hands-on Labs (To Perform Later)

## Lab 1

Create an ECS Cluster using EC2 Launch Type.

---

## Lab 2

Create an ECS Cluster using Fargate.

---

## Lab 3

Deploy the same application on both launch types.

---

## Lab 4

Measure startup time and operational differences.

---

## Lab 5

Configure a Capacity Provider backed by an Auto Scaling Group.

---

## Lab 6

Run a workload using Fargate Spot and observe its behavior.

---

# 30. One-Page Revision

```
                 ECS
                  │
      ┌───────────┴───────────┐
      │                       │
     EC2                  Fargate
      │                       │
Manage Servers          AWS Manages Servers
      │                       │
Lower Cost              Lower Operations
      │                       │
Savings Plans           Fast Deployments
      │
Capacity Providers
      │
EC2 / Fargate / Fargate Spot
```

Remember

- EC2 Launch Type
- Fargate
- Fargate Spot
- Capacity Providers
- Savings Plans
- Reserved Instances
- Cost Optimization
- Predictable vs Variable Workloads

---

# 31. Think Like a Production Engineer

Don't think:

> "Which launch type is better?"

Think:

> "Which launch type best matches the business requirements?"

Decision Flow

```
Predictable Workload?
        │
      Yes
        │
       EC2
        │
Need Simplicity?
        │
      Yes
        │
    Fargate
        │
Interruptible?
        │
      Yes
        │
 Fargate Spot
```

# End of Part 2
