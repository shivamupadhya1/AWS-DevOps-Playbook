# Amazon Elastic Kubernetes Service (EKS)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 08
>
> Part 2 – Node Groups, Autoscaling & Compute

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain Managed and Self-Managed Node Groups
- Understand EKS Fargate
- Understand Cluster Autoscaler
- Understand Karpenter
- Understand Spot Nodes
- Design cost-optimized EKS clusters
- Answer production interview questions

---

# 1. Compute Options in EKS

An EKS cluster needs compute to run Pods.

AWS provides three major compute options:

```
Managed Node Groups

↓

Self-Managed Nodes

↓

AWS Fargate
```

---

# 2. Managed Node Groups

⭐⭐⭐⭐⭐

Managed Node Groups are EC2 worker nodes managed by AWS.

AWS automates:

- Node provisioning
- Joining nodes to the cluster
- Kubernetes version updates
- Node replacement
- Health monitoring

You still manage:

- EC2 instance type
- Scaling configuration
- Labels
- Taints
- Applications

---

# 3. Architecture

```
EKS Cluster

↓

Managed Node Group

↓

Auto Scaling Group

↓

EC2 Instances

↓

Pods
```

AWS automatically creates and manages the Auto Scaling Group.

---

# 4. Advantages

- Easy upgrades
- Automatic node replacement
- AWS-managed lifecycle
- Less operational overhead
- Recommended for most workloads

---

# 5. Disadvantages

- Less customization
- Limited control over bootstrap configuration
- Not suitable for every specialized workload

---

# 6. Self-Managed Node Groups

You create and manage the EC2 instances yourself.

Responsibilities

- Launch Template
- AMI selection
- Bootstrap script
- Security patches
- Upgrades
- Scaling
- Node replacement

Architecture

```
EKS

↓

Your Auto Scaling Group

↓

EC2

↓

Pods
```

---

# 7. When to Choose Self-Managed Nodes

Use Self-Managed Nodes when you need:

- Custom AMIs
- Specialized bootstrap scripts
- GPU workloads
- Custom kernel modules
- Complete control over node configuration

---

# 8. Managed vs Self-Managed

| Feature | Managed | Self-Managed |
|----------|---------|--------------|
| Upgrades | AWS | Customer |
| Patching | AWS-assisted | Customer |
| Node Replacement | AWS | Customer |
| Customization | Medium | High |
| Operational Overhead | Low | High |

---

# 9. EKS Fargate

⭐⭐⭐⭐⭐

Fargate removes the need to manage worker nodes.

Architecture

```
Pod

↓

Fargate

↓

AWS Infrastructure
```

No EC2 instances are visible to you.

---

# 10. Fargate Profiles

A Fargate Profile defines which Pods run on Fargate.

Example

```
Namespace

↓

production

↓

Runs on Fargate
```

You can match using:

- Namespace
- Labels

---

# 11. When to Use Fargate

- Small teams
- Event-driven workloads
- Bursty traffic
- Development environments
- Serverless Kubernetes

---

# 12. When NOT to Use Fargate

Avoid Fargate when:

- Large predictable workloads
- GPU workloads
- DaemonSets are required
- Host-level customization is needed
- Cost optimization for always-on workloads is critical

---

# 13. Managed Nodes vs Fargate

| Feature | Managed Nodes | Fargate |
|----------|---------------|----------|
| EC2 Access | Yes | No |
| SSH | Yes | No |
| DaemonSets | Yes | Limited |
| Operational Effort | Medium | Low |
| Cost for 24×7 | Lower | Higher |
| Best For | Long-running apps | Variable workloads |

---

# 14. Auto Scaling in EKS

There are two different scaling mechanisms.

```
Pods

↓

Horizontal Pod Autoscaler (HPA)

------------------------------

Nodes

↓

Cluster Autoscaler / Karpenter
```

Many interview candidates confuse these.

---

# 15. Horizontal Pod Autoscaler (HPA)

Scales Pods.

Example

```
CPU

85%

↓

Pods

3

↓

6
```

HPA **does not create EC2 instances**.

---

# 16. Cluster Autoscaler

⭐⭐⭐⭐⭐

Cluster Autoscaler adds or removes worker nodes.

Flow

```
Pending Pod

↓

No Node Capacity

↓

Cluster Autoscaler

↓

Auto Scaling Group

↓

New EC2

↓

Pod Scheduled
```

---

# 17. Important Interview Point

Cluster Autoscaler works **only if** the Pod is pending because of insufficient cluster resources.

It **cannot** solve:

- ImagePullBackOff
- CrashLoopBackOff
- Invalid YAML
- Missing Secret
- Scheduling constraints unrelated to capacity

---

# 18. Example

Pod Pending

Reason

```
Insufficient CPU
```

↓

Cluster Autoscaler launches a new node.

But

```
CrashLoopBackOff
```

↓

Cluster Autoscaler does nothing.

---

# 19. Karpenter

⭐⭐⭐⭐⭐

Karpenter is AWS's modern node provisioning solution.

Instead of only increasing an existing Auto Scaling Group, Karpenter provisions the **right EC2 instance type** based on Pod requirements.

Flow

```
Pending Pod

↓

Karpenter

↓

Chooses Best EC2 Type

↓

Launches Node

↓

Schedules Pod
```

---

# 20. Cluster Autoscaler vs Karpenter

| Feature | Cluster Autoscaler | Karpenter |
|----------|-------------------|-----------|
| Uses Existing ASG | Yes | No (direct provisioning) |
| Instance Selection | Fixed | Dynamic |
| Scaling Speed | Good | Faster |
| Cost Optimization | Good | Better |
| Flexibility | Medium | High |

---

# 21. Spot Nodes

Spot Instances use unused EC2 capacity.

Benefits

- Very low cost

Risk

- AWS can terminate them with notice.

Suitable for

- CI/CD
- Batch jobs
- Stateless applications

Not ideal for

- Critical databases
- Stateful production services

---

# 22. Node Labels

Labels identify nodes.

Example

```
environment=prod

disk=ssd

gpu=true
```

Pods can target nodes using nodeSelector or node affinity.

---

# 23. Taints & Tolerations

Taints keep Pods away from nodes.

Example

```
gpu=true:NoSchedule
```

Only Pods with the matching toleration can run there.

Useful for:

- GPU nodes
- Dedicated workloads
- Isolated infrastructure

---

# 24. Production Scenario 1

Problem

Pod remains Pending.

---

### Investigation

```bash
kubectl describe pod <pod-name>
```

Look for:

```
Insufficient CPU

Insufficient Memory

No Nodes Available
```

---

### Solution

If the reason is lack of capacity:

- Cluster Autoscaler
- Karpenter
- Manual node scaling

---

# 25. Production Scenario 2

Problem

Cluster Autoscaler does not add nodes.

Possible Causes

- Incorrect IAM permissions
- ASG not tagged correctly
- Unsupported node group
- Pending reason is not resource-related

---

# 26. Production Scenario 3

Problem

Workload suddenly becomes expensive.

Investigation

- Node utilization
- Instance type
- Spot usage
- Over-provisioning
- Resource requests

---

# 27. Best Practices

- Use Managed Node Groups by default.
- Use Spot Nodes for fault-tolerant workloads.
- Configure HPA for application scaling.
- Configure Cluster Autoscaler or Karpenter for infrastructure scaling.
- Set realistic CPU and memory requests.
- Label and taint nodes intentionally.

---

# 28. Common Mistakes

❌ Assuming HPA adds EC2 instances.

---

❌ Expecting Cluster Autoscaler to fix CrashLoopBackOff.

---

❌ Running critical workloads only on Spot Nodes.

---

❌ Using oversized instance types with low utilization.

---

# 29. Interview Questions

## Question 1

Why would you choose Managed Node Groups?

### Perfect Answer

I prefer Managed Node Groups because AWS manages node lifecycle operations such as provisioning, upgrades, health monitoring, and replacement. This reduces operational overhead while still allowing me to control instance types, scaling, labels, and taints.

---

## Question 2

When would you choose Self-Managed Nodes?

### Perfect Answer

I choose Self-Managed Nodes when I need advanced customization, such as custom AMIs, specialized bootstrap scripts, GPU drivers, or kernel modifications that are not easily supported by Managed Node Groups.

---

## Question 3

What is the difference between HPA and Cluster Autoscaler?

### Perfect Answer

HPA scales the number of Pods based on application metrics such as CPU or memory. Cluster Autoscaler scales the underlying worker nodes when Pods cannot be scheduled due to insufficient resources.

---

## Question 4

When does Cluster Autoscaler work?

### Perfect Answer

Cluster Autoscaler works when Pods are pending because there is insufficient cluster capacity, such as CPU or memory. It does not solve application-level failures like CrashLoopBackOff.

---

## Question 5

What is Karpenter?

### Perfect Answer

Karpenter is an AWS-native node provisioning solution that dynamically launches the most appropriate EC2 instance types based on Pod requirements, improving scaling speed and cost optimization.

---

## Question 6

When would you use Fargate on EKS?

### Perfect Answer

I use Fargate for small teams, bursty workloads, development environments, or serverless Kubernetes use cases where minimizing infrastructure management is more important than fine-grained control.

---

## Question 7

Why shouldn't critical workloads run only on Spot Instances?

### Perfect Answer

Spot Instances can be interrupted by AWS when capacity is reclaimed. Critical workloads require more stable compute, so they should run on On-Demand or Reserved capacity, with Spot used only where interruptions are acceptable.

---

## Question 8

A Pod is Pending. What is your first troubleshooting step?

### Perfect Answer

I run:

```bash
kubectl describe pod <pod-name>
```

to identify the scheduling reason. If the issue is insufficient CPU or memory, I verify Cluster Autoscaler or Karpenter. If the reason is unrelated to capacity, I investigate that specific cause instead.

---

# 30. Amazon Cross Questions

### Question

Your company has predictable traffic 24×7. Which compute option would you choose?

### Perfect Answer

Managed Node Groups with Savings Plans or Reserved Instances because they provide lower long-term costs while reducing operational overhead.

---

### Question

Would Cluster Autoscaler solve ImagePullBackOff?

### Perfect Answer

No.

ImagePullBackOff is an image retrieval problem. Cluster Autoscaler only adds nodes when Pods cannot be scheduled due to insufficient cluster capacity.

---

### Question

Can HPA create EC2 instances?

### Perfect Answer

No.

HPA only changes the number of Pods. Worker node scaling is handled by Cluster Autoscaler or Karpenter.

---

### Question

Can you use Managed Node Groups and Fargate in the same EKS cluster?

### Perfect Answer

Yes.

An EKS cluster can run workloads on both Managed Node Groups and Fargate. Workloads are directed using scheduling rules and Fargate Profiles.

---

# 31. Hands-on Labs (To Perform Later)

## Lab 1

Create a Managed Node Group.

---

## Lab 2

Deploy a sample application.

---

## Lab 3

Install Metrics Server.

---

## Lab 4

Configure Horizontal Pod Autoscaler.

---

## Lab 5

Install Cluster Autoscaler.

---

## Lab 6

Create resource pressure and observe automatic node scaling.

---

## Lab 7

Deploy a workload to Fargate using a Fargate Profile.

---

# 32. One-Page Revision

```
               EKS Compute

                    │
     ┌──────────────┼──────────────┐
     │              │              │
Managed Nodes   Self-Managed    Fargate
     │              │              │
     └──────────────┼──────────────┘
                    │
          Pending Pod?
                    │
        Cluster Autoscaler
              or
            Karpenter
                    │
             New Worker Node
                    │
               Pod Scheduled
```

Remember

- Managed Node Groups
- Self-Managed Nodes
- Fargate
- HPA
- Cluster Autoscaler
- Karpenter
- Spot Nodes
- Labels
- Taints
- Tolerations

---

# Think Like a Production Engineer

Don't think:

> "The Pod is Pending."

Think:

> "Why is it Pending?"

Follow this order:

```
Pending Pod
      │
kubectl describe pod
      │
Resource Issue?
      │
Yes ──► Cluster Autoscaler / Karpenter
      │
No
      │
Check Events
      │
Image
Secrets
Taints
Affinity
PVC
IAM
Network
```

The **reason** determines the solution. Never assume every Pending Pod needs more nodes.

# End of Part 2
