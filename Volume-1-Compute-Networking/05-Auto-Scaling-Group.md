# Amazon Auto Scaling Group (ASG)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 05
>
> Part 1

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain why AWS introduced Auto Scaling.
- Understand Launch Templates.
- Explain Desired, Minimum, and Maximum Capacity.
- Understand the complete request flow.
- Design highly available architectures.
- Answer senior DevOps interview questions.

---

# 1. Business Problem

Imagine your company has an e-commerce website.

Normal Day

```
500 Users
```

One EC2

```
4 vCPU

16 GB RAM
```

Everything works.

---

Now

Black Friday Sale

```
500

↓

50,000 Users
```

Question

Can one EC2 handle this traffic?

No.

CPU

```
100%
```

Memory

```
95%
```

Application

```
Slow

↓

Crash
```

Now imagine

After sale

Traffic again becomes

```
500 Users
```

Question

Should we keep paying for

20 EC2 instances?

No.

AWS needed a service which could

- Automatically add servers
- Automatically remove servers
- Maintain availability
- Reduce infrastructure cost

Hence,

AWS introduced

# Auto Scaling Group (ASG)

---

# 2. What is Auto Scaling Group?

Auto Scaling Group is a service that automatically

- Launches EC2 instances
- Terminates EC2 instances
- Maintains desired capacity
- Replaces unhealthy instances

based on demand.

Think of ASG as a manager.

It doesn't run applications.

It manages EC2 instances.

---

# 3. Internal Working

```
Users

↓

ALB

↓

Target Group

↓

Auto Scaling Group

↓

Launch Template

↓

EC2
```

Notice

Auto Scaling

does not create EC2 from scratch.

It always launches EC2 using a

Launch Template.

---

# 4. Components of ASG

---

## Launch Template

This is one of the most asked interview topics.

Launch Template contains

```
AMI

↓

Instance Type

↓

Security Group

↓

IAM Role

↓

User Data

↓

EBS

↓

Network Configuration
```

Question

Can ASG launch an EC2 without Launch Template?

No.

---

## Auto Scaling Group

Contains

```
Minimum

Desired

Maximum
```

capacity.

It manages

all EC2 instances.

---

## CloudWatch

Provides metrics.

Example

```
CPU

Memory

Network

Custom Metrics
```

These metrics trigger scaling policies.

---

## Target Group

Keeps track of

Healthy

and

Unhealthy

instances.

---

## Application Load Balancer

Distributes traffic

between EC2 instances.

---

# 5. Request Flow

```
User

↓

Route53

↓

ALB

↓

Target Group

↓

ASG

↓

EC2

↓

Application
```

If

CPU increases

↓

CloudWatch Alarm

↓

Scaling Policy

↓

ASG

↓

Launch Template

↓

New EC2

↓

Register with Target Group

↓

ALB starts routing traffic
```

---

# 6. Desired Capacity

Suppose

```
Desired = 3
```

ASG always tries to keep

```
3 Running EC2
```

If

one EC2 crashes

```
3

↓

2
```

ASG automatically launches

one more EC2.

Desired again becomes

```
3
```

---

# 7. Minimum Capacity

Example

```
Minimum = 2
```

Even if

traffic becomes zero

ASG never goes below

```
2 EC2
```

---

# 8. Maximum Capacity

Example

```
Maximum = 10
```

If CPU becomes

```
100%
```

ASG can launch

new EC2

until

```
10 Instances
```

After that

Scaling stops.

---

# 9. Capacity Example

```
Minimum = 2

Desired = 4

Maximum = 10
```

Current

```
4 Running EC2
```

Traffic increases.

CloudWatch Alarm triggers.

ASG launches

```
5

↓

6

↓

7

↓

8
```

Until

load decreases

or

Maximum

is reached.

---

# 10. Health Checks

ASG supports

---

## EC2 Health Check

Checks

```
Instance Status
```

If EC2 fails

Replace.

---

## ELB Health Check

Checks

```
Application Health
```

Example

```
/health
```

If endpoint fails

Instance replaced.

Production always uses

ELB Health Checks.

---

# 11. Real Production Story

A payment application

always maintained

```
Desired = 6
```

One EC2 crashed

at midnight.

Nobody noticed.

Why?

Because

ASG detected

```
6

↓

5
```

Immediately

```
Launch Template

↓

New EC2

↓

Healthy

↓

Desired = 6
```

Application remained available.

---

# 12. Best Practices

✅ Use Launch Templates.

✅ Never manually launch production EC2.

✅ Use ALB.

✅ Enable ELB Health Checks.

✅ Keep Desired Capacity greater than one.

✅ Use IAM Roles.

✅ Use CloudWatch monitoring.

---

# 13. Common Mistakes

❌ Desired = 1

One EC2 failure

↓

Application Down.

---

❌ Maximum = Desired

System

cannot scale.

---

❌ Using public AMIs.

Use

Golden AMIs.

---

❌ Not using Health Checks.

---

# 14. Interview Questions

---

## Question 1

Why did AWS introduce Auto Scaling?

### Perfect Answer

AWS introduced Auto Scaling to automatically adjust the number of EC2 instances based on application demand. This improves availability during traffic spikes, reduces infrastructure cost during low traffic, and eliminates manual intervention for scaling operations.

---

## Question 2

Difference between Desired, Minimum and Maximum Capacity?

### Perfect Answer

Minimum Capacity defines the lowest number of instances ASG will maintain.

Desired Capacity is the target number of running instances.

Maximum Capacity limits how many instances ASG can launch during scale-out.

Example:

```
Minimum = 2

Desired = 4

Maximum = 10
```

Normally

4 EC2 run.

ASG can never go below 2

and never above 10.

---

## Question 3

What happens if one EC2 crashes?

### Perfect Answer

ASG continuously monitors instance health. If an instance becomes unhealthy based on EC2 or ELB health checks, ASG terminates it and launches a replacement instance using the Launch Template to maintain the desired capacity.

---

## Question 4

Can ASG launch EC2 without AMI?

### Perfect Answer

No.

Every EC2 launched by ASG is created from the Launch Template, which references an AMI. Without an AMI, ASG cannot provision a new instance.

---

## Question 5

Can we manually terminate an EC2 inside an Auto Scaling Group?

### Perfect Answer

Yes.

However,

ASG detects

```
Desired = 3

Running = 2
```

and automatically launches a new EC2 to restore the desired capacity.

---

# 15. Amazon Cross Questions

---

### Question

Why does ASG use Launch Templates instead of configuring servers after launch?

### Perfect Answer

Using Launch Templates ensures every EC2 instance is created with identical configuration, including the operating system, instance type, IAM role, security groups, user data, and storage settings. This provides consistency, faster provisioning, and predictable deployments.

---

### Question

Can ASG work without an ALB?

### Perfect Answer

Yes.

ASG can function independently and maintain the desired number of EC2 instances. However, for web applications, an ALB is typically used to distribute traffic and perform application-level health checks.

---

# 16. Think Like a Production Engineer

Don't think

> ASG launches EC2.

Think

> ASG maintains application availability.

Its real responsibility is

```
Desired Capacity

↓

Healthy Instances

↓

Business Continuity
```

---

# Revision Sheet

Remember

- ASG manages EC2.
- Uses Launch Templates.
- Maintains Desired Capacity.
- Uses CloudWatch metrics.
- Supports EC2 and ELB health checks.
- Replaces unhealthy instances automatically.

```
Traffic ↑

↓

CloudWatch

↓

Scaling Policy

↓

ASG

↓

Launch Template

↓

New EC2

↓

Target Group

↓

ALB
```

---

# End of Part 1
