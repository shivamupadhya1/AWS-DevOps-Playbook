# Amazon ECS (Elastic Container Service)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 07
>
> Part 1 - ECS Fundamentals

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain Amazon ECS architecture
- Explain every ECS component
- Understand Task Definition
- Understand Task vs Service
- Understand ECS Cluster
- Understand ECS Agent
- Understand ECS Scheduler
- Explain ECS workflow
- Troubleshoot ECS basics
- Answer ECS interview questions confidently

---

# 1. What is Amazon ECS?

Amazon ECS (Elastic Container Service) is AWS's fully managed container orchestration service.

Its responsibility is to:

- Deploy containers
- Schedule containers
- Restart failed containers
- Scale applications
- Integrate with AWS services

Think of ECS as a manager that decides:

> Which container should run?
>
> Where should it run?
>
> How many copies should run?
>
> What should happen if one crashes?

---

# 2. Why ECS?

Without ECS

```
EC2

↓

Install Docker

↓

Run docker run

↓

Monitor Container

↓

Restart Container

↓

Scale Container

↓

Load Balancer
```

Everything is manual.

With ECS

```
Application

↓

Task Definition

↓

ECS

↓

Containers Running
```

AWS manages orchestration.

---

# 3. ECS Architecture

```
Developer

↓

Docker Build

↓

Amazon ECR

↓

Task Definition

↓

ECS Service

↓

ECS Scheduler

↓

ECS Cluster

↓

EC2 / Fargate

↓

Container
```

This is the complete deployment flow.

---

# 4. ECS Components

Major ECS Components

```
Cluster

Task Definition

Task

Service

Container

ECS Agent

Scheduler

Capacity Provider
```

We'll study each.

---

# 5. ECS Cluster

⭐⭐⭐⭐⭐

Cluster is a logical grouping of compute resources.

Example

```
Production Cluster

↓

EC2-1

EC2-2

EC2-3
```

OR

```
Production Cluster

↓

Fargate
```

A cluster itself does not run applications.

It only provides capacity.

Think of a cluster as:

> A parking lot.

Cars (Tasks) park inside it.

---

# 6. Container Instance (EC2 Launch Type)

If using EC2 Launch Type

```
Cluster

↓

EC2 Instance

↓

Docker

↓

Container
```

The EC2 instance registered with ECS is called a **Container Instance**.

Requirements

- ECS Agent installed
- IAM Role attached
- Docker running

---

# 7. ECS Agent

⭐⭐⭐⭐⭐

The ECS Agent runs on every EC2 Container Instance.

Responsibilities

- Registers EC2 with ECS
- Receives tasks
- Starts containers
- Stops containers
- Sends health status
- Reports resource usage

Architecture

```
AWS ECS

↓

ECS Agent

↓

Docker

↓

Container
```

Without ECS Agent

No task can run.

---

# 8. Task Definition

⭐⭐⭐⭐⭐

Task Definition is the blueprint of your application.

It contains

- Docker Image
- CPU
- Memory
- Port
- Environment Variables
- Secrets
- IAM Role
- Logging
- Volumes

Example

```
Image

↓

nginx

CPU

↓

512

Memory

↓

1024

Port

↓

80
```

Every change creates a new revision.

Example

```
Revision 1

↓

Revision 2

↓

Revision 3
```

Task Definitions are immutable.

---

# 9. Task

⭐⭐⭐⭐⭐

A Task is a **running instance of a Task Definition**.

Exactly like

```
Docker Image

↓

Container
```

Similarly

```
Task Definition

↓

Task
```

One Task Definition

↓

Many Tasks

```
Task Definition

↓

Task 1

Task 2

Task 3
```

---

# 10. Service

⭐⭐⭐⭐⭐

Service ensures that the desired number of tasks is always running.

Example

Desired Count

```
3
```

Current Running

```
2
```

ECS automatically creates one more task.

Architecture

```
Service

↓

Task 1

Task 2

Task 3
```

---

# 11. Desired Count

Suppose

Desired Count

```
5
```

Running

```
4
```

ECS creates

```
Task 5
```

If one task crashes

```
Task 3

↓

Stopped
```

Service immediately launches another.

---

# 12. ECS Scheduler

⭐⭐⭐⭐⭐

Scheduler decides

- Which EC2 should run the task?
- Does enough CPU exist?
- Does enough Memory exist?

Architecture

```
Service

↓

Scheduler

↓

Cluster

↓

EC2 Selected
```

---

# 13. ECS Control Plane

AWS manages

- Scheduling
- APIs
- Cluster State
- Deployments
- Health Monitoring

Users do not manage this.

---

# 14. ECS Workflow

Complete Flow

```
Developer

↓

docker build

↓

Push to ECR

↓

Create Task Definition

↓

Create Service

↓

Scheduler

↓

Cluster

↓

Container Running
```

---

# 15. ECS Deployment Lifecycle

```
Task Definition

↓

Task Created

↓

Image Pulled

↓

Container Started

↓

Health Check

↓

Running
```

---

# 16. ECS with ECR

```
Developer

↓

Docker Image

↓

Amazon ECR

↓

Task Definition

↓

ECS Task

↓

Container Running
```

---

# 17. ECS Health Monitoring

ECS continuously monitors

- Running
- Pending
- Stopped
- Failed

If a Service is managing the Task,

it replaces failed tasks automatically.

---

# 18. Production Example

Suppose your application has

Frontend

Backend

Redis

Each becomes a separate ECS Service.

```
ALB

↓

Frontend Service

↓

Backend Service

↓

Redis Service
```

Each Service has its own Task Definition.

---

# 19. Difference Between Task and Service

Task

- Runs once
- May stop
- Not restarted automatically

Service

- Maintains desired count
- Restarts failed tasks
- Supports deployments
- Used for long-running applications

---

# 20. Difference Between Task Definition and Task

Task Definition

Blueprint

Task

Running application

Exactly like

Docker Image vs Docker Container.

---

# 21. Production Scenario 1

Problem

One container crashes.

Question

What happens?

Answer

If started through a Service,

ECS launches another Task automatically.

If started as a standalone Task,

it remains stopped until someone manually starts it again.

---

# 22. Production Scenario 2

Problem

Task remains in PENDING.

Possible Reasons

- CPU unavailable
- Memory unavailable
- Image pull failure
- IAM issue
- Networking issue

(We'll study troubleshooting in detail later.)

---

# 23. Production Scenario 3

Problem

Task stopped immediately.

Possible Causes

- Application crashed
- Wrong CMD
- Health check failed
- Missing environment variables

Check

```
Stopped Reason

CloudWatch Logs

Container Exit Code
```

---

# 24. Best Practices

- Keep one application per Task.
- Use Services for production workloads.
- Store images in ECR.
- Enable CloudWatch Logs.
- Version Task Definitions.
- Use immutable image tags.
- Use Task Roles instead of embedding AWS credentials.

---

# 25. Common Mistakes

❌ Running production workloads as standalone Tasks.

Use Services.

---

❌ Using mutable `latest` image tags.

---

❌ Ignoring Task Definition revisions.

---

❌ Not enabling logging.

---

# 26. Interview Questions

## Question 1

What is Amazon ECS?

### Perfect Answer

Amazon ECS is a fully managed container orchestration service that schedules, runs, monitors, and scales Docker containers on either EC2 instances or AWS Fargate.

---

## Question 2

What is a Cluster?

### Perfect Answer

A Cluster is a logical grouping of compute resources where ECS schedules Tasks. It can contain EC2 instances or use AWS Fargate as the compute engine.

---

## Question 3

What is a Task Definition?

### Perfect Answer

A Task Definition is a versioned blueprint that defines how one or more containers should run, including image, CPU, memory, networking, IAM roles, environment variables, logging, and volumes.

---

## Question 4

Difference between Task Definition and Task?

### Perfect Answer

A Task Definition is the configuration template, while a Task is the running instance created from that template. It is similar to the relationship between a Docker image and a Docker container.

---

## Question 5

What is a Service?

### Perfect Answer

An ECS Service maintains the desired number of running Tasks, automatically replacing failed Tasks and supporting deployments such as rolling updates.

---

## Question 6

What happens if an ECS Task fails?

### Perfect Answer

If the Task is managed by an ECS Service, ECS automatically launches a replacement Task to maintain the configured desired count. Standalone Tasks are not restarted automatically.

---

## Question 7

What does the ECS Agent do?

### Perfect Answer

The ECS Agent runs on EC2 container instances. It registers the instance with ECS, receives Task instructions from the ECS control plane, starts and stops containers, and reports status and resource information.

---

## Question 8

Can ECS work without Docker?

### Perfect Answer

ECS requires a compatible container runtime. Traditionally this was Docker, while modern ECS environments may use containerd under the hood depending on the platform. The user experience remains the same—ECS runs OCI-compatible containers.

---

## Question 9

Why does every Task Definition create a new revision?

### Perfect Answer

Task Definitions are immutable. Any change—such as a new image version, CPU value, environment variable, or memory allocation—creates a new revision, allowing controlled deployments and rollbacks.

---

## Question 10

Can multiple Services use the same Task Definition?

### Perfect Answer

Yes.

Multiple Services can use the same Task Definition revision, each with its own desired count, deployment settings, and load balancer configuration.

---

# 27. Amazon Cross Questions

### Question

Can one Cluster contain both EC2 and Fargate?

### Perfect Answer

Yes.

A single ECS Cluster can support both EC2 and Fargate capacity, provided the Services or Tasks are configured to use the appropriate launch type or capacity provider.

---

### Question

Can one Task Definition contain multiple containers?

### Perfect Answer

Yes.

A Task Definition can define multiple containers that run together as a single Task, such as an application container and a sidecar container for logging or monitoring.

---

### Question

Does deleting a Service delete its Task Definition?

### Perfect Answer

No.

Deleting a Service stops its Tasks, but the Task Definition revisions remain available unless explicitly deregistered.

---

### Question

Can you run a Task without creating a Service?

### Perfect Answer

Yes.

Standalone Tasks are useful for one-time jobs, batch processing, or scheduled workloads. Long-running applications should generally be deployed using Services.

---

# 28. Hands-on Labs (To Perform Later)

## Lab 1

Create an ECS Cluster.

---

## Lab 2

Create an ECR repository.

---

## Lab 3

Push an Nginx image to ECR.

---

## Lab 4

Create a Task Definition using the ECR image.

---

## Lab 5

Run a standalone Task.

---

## Lab 6

Create an ECS Service with Desired Count = 2.

---

## Lab 7

Stop one Task manually and observe ECS automatically launching a replacement.

---

# 29. One-Page Revision

```
Docker Image
      │
      ▼
Amazon ECR
      │
      ▼
Task Definition
      │
      ▼
Task
      │
      ▼
Service
      │
      ▼
Scheduler
      │
      ▼
Cluster
      │
      ▼
EC2 / Fargate
```

Remember

- Cluster
- Task Definition
- Task
- Service
- Scheduler
- ECS Agent
- Desired Count
- Control Plane
- ECR Integration

---

# 30. Think Like a Production Engineer

Don't think:

> "ECS runs containers."

Think:

> "ECS is responsible for the entire lifecycle of containerized workloads—from scheduling and deployment to monitoring and self-healing."

Whenever someone says:

```
Task Not Running

↓

Service

↓

Scheduler

↓

Cluster Capacity

↓

ECR

↓

Logs
```

That should become your default troubleshooting flow.

# End of Part 1
