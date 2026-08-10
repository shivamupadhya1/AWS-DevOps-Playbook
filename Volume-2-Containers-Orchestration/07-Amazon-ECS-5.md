# Amazon ECS (Elastic Container Service)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 07
>
> Part 5 – Production Troubleshooting

---

# Chapter Objective

After completing this chapter, you should be able to:

- Debug ECS production issues
- Identify root causes quickly
- Follow a structured troubleshooting flow
- Explain your approach confidently in interviews
- Handle real-world production incidents

---

# Universal ECS Troubleshooting Flow

Whenever an ECS issue occurs, follow this order:

```
Problem

↓

ECS Service Events

↓

Task Status

↓

Task Stopped Reason

↓

CloudWatch Logs

↓

Target Group Health

↓

ALB Health Checks

↓

ECR Image

↓

IAM Permissions

↓

Networking

↓

Root Cause

↓

Fix
```

Never start guessing.

---

# Scenario 1

## Problem

Task is stuck in **PENDING**.

---

### Possible Reasons

- No CPU available
- No memory available
- No EC2 capacity
- Capacity Provider issue
- Image pull delay
- ENI allocation issue (Fargate)
- Placement constraints

---

### Investigation

```bash
aws ecs describe-services
```

```bash
aws ecs describe-tasks
```

Console

```
Cluster

↓

Service

↓

Events
```

Look for messages like:

```
insufficient CPU

insufficient memory

unable to place task
```

---

### Root Cause

Scheduler cannot place the Task.

---

### Fix

- Scale the cluster
- Add EC2 instances
- Increase Fargate capacity if applicable
- Reduce Task CPU or memory
- Review placement constraints

---

### Interview Answer

"I would first inspect ECS Service Events because they usually explain why the scheduler could not place the Task. Based on the event message, I would verify cluster capacity, CPU, memory, networking, and placement constraints before making changes."

---

# Scenario 2

## Problem

Task immediately stops after starting.

---

### Possible Reasons

- Application crash
- Wrong CMD/ENTRYPOINT
- Missing environment variables
- Secret retrieval failure
- Application startup exception

---

### Investigation

Check

```
Stopped Reason

Exit Code

CloudWatch Logs
```

---

### Fix

Correct the application or Task Definition, create a new revision, and redeploy.

---

### Interview Answer

"My first step is checking the container exit code and CloudWatch Logs because they reveal why the application terminated."

---

# Scenario 3

## Problem

CannotPullContainerError

---

### Possible Reasons

- Image missing
- Wrong tag
- Repository deleted
- IAM permission missing
- Authentication failure
- Network issue

---

### Investigation

Verify

- Repository exists
- Image tag exists
- Task Execution Role
- VPC connectivity
- NAT Gateway (private subnets)
- VPC Endpoint for ECR (if used)

---

### Fix

Correct the image reference or IAM configuration and redeploy.

---

### Interview Answer

"I would verify the Task Definition image URI, confirm the image exists in ECR, validate the Task Execution Role permissions, and ensure the Task has network connectivity to ECR."

---

# Scenario 4

## Problem

New image pushed.

ECS still runs the old version.

---

### Possible Reasons

- Service not updated
- Task Definition not revised
- Mutable `latest` tag
- Deployment not triggered

---

### Investigation

Compare

```
Running Task Definition Revision

↓

Latest Task Definition Revision
```

---

### Fix

Register a new Task Definition revision and update the Service.

---

### Interview Answer

"Pushing a new image alone does not update running Tasks. ECS deploys only when the Service is updated to a new Task Definition revision."

---

# Scenario 5

## Problem

ALB returns **502 Bad Gateway**.

---

### Possible Reasons

- Wrong container port
- Wrong Target Group port
- Application crashed
- Security Group blocks traffic
- Health check misconfiguration

---

### Investigation

Check

```
ALB

↓

Target Group

↓

Healthy Targets

↓

Container Port

↓

Application Logs
```

---

### Fix

Correct the port mapping, health check path, or networking.

---

### Interview Answer

"I would inspect Target Group health first, then verify the container port, security groups, and application logs."

---

# Scenario 6

## Problem

ALB returns **503 Service Unavailable**.

---

### Possible Reasons

- No healthy Tasks
- Desired Count = 0
- Deployment failed
- Health checks failing

---

### Investigation

```
Target Group

↓

Healthy Hosts

↓

Service Events
```

---

### Fix

Restore healthy Tasks and resolve deployment issues.

---

# Scenario 7

## Problem

Deployment never finishes.

---

### Possible Reasons

- Health checks never pass
- Application startup takes too long
- CPU shortage
- Memory shortage
- Broken application

---

### Investigation

```
Service Events

↓

CloudWatch Logs

↓

Target Group Health
```

---

### Fix

Correct the startup issue and redeploy.

---

# Scenario 8

## Problem

Service keeps replacing Tasks.

---

### Possible Reasons

- Health check failure
- Application crash
- Memory exhaustion
- OOMKilled
- Exit code failure

---

### Investigation

Check

```
Task History

↓

CloudWatch Logs

↓

Exit Code
```

---

### Fix

Resolve the application or infrastructure issue causing repeated failures.

---

# Scenario 9

## Problem

CPU utilization remains above 90%.

---

### Investigation

CloudWatch Metrics

↓

Container Insights

↓

Application profiling

↓

CPU-intensive processes

---

### Fix

- Scale out Tasks
- Optimize application
- Increase CPU allocation

---

# Scenario 10

## Problem

Memory utilization reaches 100%.

---

### Investigation

CloudWatch

↓

Application logs

↓

Memory leak analysis

---

### Fix

Increase memory allocation or fix the memory leak.

---

# Scenario 11

## Problem

CloudWatch shows healthy infrastructure, but users report slowness.

---

### Investigation

- Database latency
- External API latency
- Thread pool exhaustion
- Application logs
- X-Ray tracing (if enabled)

---

### Lesson

Infrastructure metrics alone do not always explain application performance.

---

# Scenario 12

## Problem

Task cannot access S3.

---

### Investigation

Verify

```
Task Role

↓

IAM Policy

↓

Bucket Policy
```

---

### Root Cause

Task Role lacks permission.

---

### Fix

Grant least-privilege access to the Task Role.

---

# Scenario 13

## Problem

Task cannot pull Secrets Manager secret.

---

### Investigation

Verify

- Secret ARN
- Task Role permissions
- KMS permissions
- Region

---

### Fix

Correct IAM permissions and redeploy.

---

# Scenario 14

## Problem

Tasks remain healthy but traffic is uneven.

---

### Investigation

- ALB algorithm
- Sticky sessions
- Slow instances
- Request distribution

---

### Fix

Review load-balancing configuration.

---

# Scenario 15

## Problem

Cluster has no remaining capacity.

---

### Investigation

```
Available CPU

↓

Available Memory

↓

EC2 Instances

↓

Auto Scaling Group
```

---

### Fix

Scale the Auto Scaling Group or reduce Task resource requirements.

---

# Scenario 16

## Problem

ECS Agent disconnected.

---

### Investigation

On EC2

```bash
systemctl status ecs
```

Check

- ECS Agent
- Docker
- IAM Instance Profile
- Network connectivity

---

### Fix

Restart the ECS Agent or repair the EC2 instance.

---

# Scenario 17

## Problem

Deployment succeeds.

Application still fails.

---

### Investigation

- Business logs
- Database connectivity
- Third-party APIs
- Configuration
- Secrets

---

### Lesson

A successful deployment does not guarantee a healthy application.

---

# Scenario 18

## Problem

Task starts but exits with code 137.

---

### Root Cause

Usually indicates the container was killed due to insufficient memory (OOM).

---

### Fix

Increase memory allocation or optimize application memory usage.

---

# Scenario 19

## Problem

Task is healthy internally but ALB marks it unhealthy.

---

### Possible Reasons

- Wrong health check path
- Wrong health check port
- Security Group issue
- Timeout
- Health check threshold

---

### Fix

Align ALB health check configuration with the application.

---

# Scenario 20

## Problem

Deployment rolled back automatically.

---

### Investigation

- Deployment alarms
- Health check failures
- CloudWatch metrics
- Application logs

---

### Fix

Correct the issue before attempting another deployment.

---

# Production Debugging Checklist

Always verify:

- ECS Service Events
- Running Task Definition
- Task Status
- Exit Code
- CloudWatch Logs
- Target Group Health
- ALB Listener
- Container Port
- Security Groups
- IAM Roles
- ECR Image Tag
- Cluster Capacity
- CPU
- Memory
- Secrets
- Environment Variables

---

# Common AWS Error Mapping

| Error | Start Troubleshooting From |
|--------|----------------------------|
| CannotPullContainerError | ECR, IAM, Network |
| Task Pending | Cluster Capacity, Scheduler |
| Task Stopped | Exit Code, Logs |
| 502 Bad Gateway | Target Group, Ports, App |
| 503 Service Unavailable | Healthy Targets, Service |
| High CPU | CloudWatch, Container Insights |
| High Memory | Memory Metrics, Logs |
| Access Denied | IAM Role |
| Secret Retrieval Failed | Secrets Manager, IAM |
| Deployment Failed | Service Events |

---

# Amazon Interview Questions

## Question 1

A Task is in PENDING. What is your approach?

### Perfect Answer

"I first inspect ECS Service Events because they usually explain why scheduling failed. Then I verify cluster CPU, memory, placement constraints, networking, and Capacity Providers before deciding whether additional infrastructure is required."

---

## Question 2

A deployment succeeds but users still receive errors. What would you do?

### Perfect Answer

"I verify Target Group health, review CloudWatch metrics and logs, check application health endpoints, validate database connectivity, and confirm the new Task Definition contains the expected configuration."

---

## Question 3

Why is ECS still running the old image?

### Perfect Answer

"ECS does not automatically deploy new images when they are pushed to ECR. A new Task Definition revision must be created and the ECS Service must be updated to use that revision."

---

## Question 4

What is the first place you check during an ECS incident?

### Perfect Answer

"I begin with ECS Service Events because they provide immediate insight into scheduling failures, deployment issues, and Task placement problems."

---

## Think Like a Production Engineer

Never troubleshoot randomly.

Always follow a structured flow:

```
Problem

↓

Service Events

↓

Task

↓

Logs

↓

ALB

↓

IAM

↓

Network

↓

Root Cause

↓

Fix

↓

Validation
```

A good DevOps engineer doesn't guess.

A good DevOps engineer follows evidence.

# End of Part 5
