# Amazon ECS (Elastic Container Service)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 07
>
> Part 3 – Deployments, Load Balancing & Scaling

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand ECS deployment strategies
- Configure ALB integration
- Understand Target Groups
- Configure Health Checks
- Explain Service Discovery
- Configure Auto Scaling
- Use CloudWatch with ECS
- Troubleshoot production deployment failures
- Answer deployment interview questions

---

# 1. ECS Deployment Flow

A production deployment follows this sequence:

```
Developer

↓

Git Push

↓

CI/CD Pipeline

↓

Docker Build

↓

Push Image to ECR

↓

Register New Task Definition Revision

↓

Update ECS Service

↓

Scheduler Starts New Tasks

↓

Health Checks

↓

Traffic Shift

↓

Old Tasks Drained

↓

Deployment Complete
```

---

# 2. ECS Deployment Types

ECS supports multiple deployment approaches.

```
Rolling Update

Blue/Green

External (Custom)
```

Canary is not a built-in ECS deployment controller but can be implemented using **ALB weighted target groups**, **AWS CodeDeploy**, or **service mesh** solutions.

---

# 3. Rolling Update

⭐⭐⭐⭐⭐

Default deployment strategy.

Example

Current

```
Task A

Task B

Task C
```

Deploy New Version

```
Task A v2

Task B

Task C
```

↓

Health Check

↓

```
Task A v2

Task B v2

Task C
```

↓

Health Check

↓

```
Task A v2

Task B v2

Task C v2
```

Advantages

- Simple
- No duplicate environment
- Default ECS deployment

Disadvantages

- Rollback affects live environment
- Deployment issues may impact production until rollback completes

---

# 4. Deployment Configuration

Two important settings

```
Minimum Healthy Percent

Maximum Percent
```

Example

Desired Count = 4

Minimum Healthy = 50%

Maximum = 200%

Meaning

ECS can temporarily run up to

```
8 Tasks
```

during deployment while ensuring at least

```
2 Healthy Tasks
```

remain available.

---

# 5. Blue/Green Deployment

⭐⭐⭐⭐⭐

Current

```
Blue Environment
```

New Version

```
Green Environment
```

Architecture

```
Users

↓

ALB

↓

Blue

Green
```

Traffic

Initially

```
100%

↓

Blue
```

After validation

```
100%

↓

Green
```

Advantages

- Near-zero downtime
- Easy rollback
- Full validation before cutover

Disadvantages

- Higher infrastructure cost
- Requires duplicate environments

---

# 6. Canary Deployment

⭐⭐⭐⭐⭐

Traffic shifts gradually.

Example

```
10%

↓

New Version
```

↓

Observe metrics

↓

```
30%
```

↓

```
50%
```

↓

```
100%
```

Advantages

- Lower deployment risk
- Detects issues early
- Limits user impact

Disadvantages

- More complex routing
- Requires traffic management

---

# 7. Which Deployment Strategy Should You Choose?

| Requirement | Strategy |
|-------------|----------|
| Simple updates | Rolling |
| Zero downtime | Blue/Green |
| Risk reduction | Canary |

---

# 8. ECS Service Discovery

Instead of hardcoding IPs,

Applications communicate using DNS.

```
Frontend

↓

backend.internal
```

↓

Backend Service

AWS Cloud Map provides service discovery.

Advantages

- No hardcoded IPs
- Automatic registration
- Easier scaling

---

# 9. ALB Integration

⭐⭐⭐⭐⭐

Architecture

```
Internet

↓

Application Load Balancer

↓

Target Group

↓

ECS Service

↓

Tasks
```

ALB distributes traffic across healthy Tasks.

---

# 10. Target Groups

A Target Group contains registered ECS Tasks.

Example

```
Target Group

↓

Task 1

Task 2

Task 3
```

ALB sends requests only to healthy targets.

---

# 11. Health Checks

ALB continuously checks

```
GET /health
```

Example

Healthy

```
HTTP 200
```

Unhealthy

```
HTTP 500

HTTP 404

Timeout
```

Unhealthy Tasks stop receiving traffic.

---

# 12. Connection Draining

Before stopping a Task

ALB waits for existing connections to finish.

This prevents users from seeing dropped requests.

---

# 13. ECS Auto Scaling

Two levels

```
Service Auto Scaling

Cluster Capacity Scaling
```

---

# 14. Service Auto Scaling

Scales Task count.

Example

CPU

```
80%
```

↓

Desired Count

```
3

↓

6
```

---

# 15. Cluster Auto Scaling

Scales EC2 capacity.

Example

Cluster

```
No CPU Remaining
```

↓

Auto Scaling Group

↓

Launch New EC2

↓

Scheduler Places Tasks

---

# 16. CloudWatch Integration

ECS publishes metrics such as:

- CPUUtilization
- MemoryUtilization
- RunningTaskCount
- PendingTaskCount

CloudWatch Alarms can trigger Auto Scaling.

---

# 17. ECS Exec

Allows secure shell-like access into running containers.

Example

```bash
aws ecs execute-command \
--cluster prod \
--task abc123 \
--container app \
--interactive \
--command "/bin/sh"
```

Useful for debugging production containers without SSH access to hosts.

---

# 18. Production Scenario 1

Problem

Deployment fails.

Check

- Task Definition
- Container logs
- Health Check
- Image version
- Environment variables
- IAM permissions

---

# 19. Production Scenario 2

Problem

ALB returns 502.

Possible Reasons

- Container not listening on expected port
- Wrong Target Group port
- Application crashed
- Health Check path incorrect
- Security Group issue

---

# 20. Production Scenario 3

Problem

ALB returns 503.

Possible Reasons

- No healthy targets
- Service Desired Count = 0
- All Tasks failed health checks
- ECS deployment failed

---

# 21. Production Scenario 4

Problem

Rolling deployment never completes.

Check

- Health Check failures
- CPU/Memory availability
- Deployment configuration
- Application startup time
- CloudWatch Logs

---

# 22. Production Scenario 5

Problem

New Task repeatedly stops.

Check

- Exit Code
- CloudWatch Logs
- Environment variables
- Secrets
- Image version

---

# 23. Best Practices

- Use ALB for production services.
- Configure meaningful health checks.
- Avoid using `/` as a health endpoint if it performs expensive work.
- Enable CloudWatch Logs.
- Use immutable image tags.
- Test deployments in staging.
- Configure deployment alarms.
- Monitor Target Group health.

---

# 24. Common Mistakes

❌ Health Check path returns 404.

---

❌ Wrong container port.

---

❌ Desired Count set to zero accidentally.

---

❌ Using mutable `latest` tags.

---

❌ Ignoring CloudWatch alarms.

---

# 25. Interview Questions

## Question 1

What is the default ECS deployment strategy?

### Perfect Answer

Rolling Update. ECS gradually replaces old Tasks with new Tasks while maintaining the configured minimum healthy percentage.

---

## Question 2

Explain Blue/Green Deployment.

### Perfect Answer

Blue/Green deployment runs two complete environments simultaneously. Users initially access the Blue environment. After validating the Green environment, traffic is switched to Green, enabling quick rollback if required.

---

## Question 3

Explain Canary Deployment.

### Perfect Answer

Canary deployment releases a new application version to a small percentage of users first. If monitoring shows healthy behavior, traffic is gradually increased until all users are on the new version.

---

## Question 4

How does ECS integrate with ALB?

### Perfect Answer

An ECS Service registers its Tasks with an ALB Target Group. The ALB performs health checks and distributes incoming traffic only to healthy Tasks.

---

## Question 5

What happens if a Task fails a health check?

### Perfect Answer

The ALB marks the Task as unhealthy and stops routing traffic to it. If the Task belongs to an ECS Service, ECS launches a replacement Task to maintain the desired count.

---

## Question 6

Difference between Service Auto Scaling and Cluster Auto Scaling?

### Perfect Answer

Service Auto Scaling changes the number of running Tasks based on metrics such as CPU or memory utilization. Cluster Auto Scaling adjusts the underlying EC2 capacity to ensure there are enough resources for those Tasks.

---

## Question 7

Why use Service Discovery?

### Perfect Answer

Service Discovery allows applications to communicate using DNS names instead of hardcoded IP addresses. This simplifies scaling and reduces configuration changes.

---

## Question 8

What causes ALB 502 errors?

### Perfect Answer

Common causes include incorrect container ports, application crashes, invalid health check configuration, backend connection failures, or networking issues between the ALB and ECS Tasks.

---

## Question 9

What causes ALB 503 errors?

### Perfect Answer

A 503 typically indicates that the ALB has no healthy backend targets. This can happen when all Tasks fail health checks, Desired Count is zero, or deployments fail.

---

## Question 10

How do you investigate a failed ECS deployment?

### Perfect Answer

I verify the Task Definition revision, inspect ECS Service events, review CloudWatch Logs, check Target Group health, validate health check configuration, confirm IAM permissions, and ensure the container image and environment variables are correct.

---

# 26. Amazon Cross Questions

### Question

Would you choose Rolling Update or Blue/Green for a banking application?

### Perfect Answer

For critical banking workloads, I would generally prefer Blue/Green because it provides safer deployments, near-zero downtime, and rapid rollback capabilities.

---

### Question

If a new deployment causes increased error rates, what would you do?

### Perfect Answer

I would stop or roll back the deployment, analyze CloudWatch metrics and application logs, identify the root cause, fix the issue, create a new Task Definition revision, and redeploy after validation.

---

### Question

Why are health checks important?

### Perfect Answer

Health checks ensure that traffic is routed only to healthy Tasks, improving application availability and preventing users from reaching failed instances.

---

### Question

Would you expose ECS Tasks directly to the Internet?

### Perfect Answer

No.

Production deployments should generally place ECS Tasks in private subnets and expose them through an Application Load Balancer for security and traffic management.

---

# 27. Hands-on Labs (To Perform Later)

## Lab 1

Deploy an ECS Service behind an ALB.

---

## Lab 2

Configure a custom `/health` endpoint.

---

## Lab 3

Perform a Rolling Update.

---

## Lab 4

Perform a Blue/Green deployment using CodeDeploy.

---

## Lab 5

Implement a Canary deployment using weighted routing.

---

## Lab 6

Configure Service Auto Scaling based on CPU utilization.

---

## Lab 7

Intentionally break the health check and observe ECS replacing unhealthy Tasks.

---

# 28. One-Page Revision

```
Git Push
   │
Docker Build
   │
Amazon ECR
   │
Task Definition
   │
ECS Service
   │
Application Load Balancer
   │
Target Group
   │
Healthy Tasks
```

Remember

- Rolling Update
- Blue/Green
- Canary
- Target Groups
- Health Checks
- Service Discovery
- Service Auto Scaling
- Cluster Auto Scaling
- CloudWatch
- ECS Exec

---

# 29. Think Like a Production Engineer

Don't think:

> "Deployment succeeded because the pipeline finished."

Think:

> "A deployment is successful only after the new Tasks are healthy, the ALB routes traffic correctly, CloudWatch metrics remain normal, and users experience no disruption."

Production Troubleshooting Flow

```
Deployment Failed
        │
Service Events
        │
Task Status
        │
Container Logs
        │
Target Group Health
        │
ALB Health Checks
        │
CloudWatch Metrics
        │
Root Cause
        │
Fix & Redeploy
```

# End of Part 3
