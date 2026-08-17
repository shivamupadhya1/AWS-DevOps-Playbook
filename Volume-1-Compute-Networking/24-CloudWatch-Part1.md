# Amazon CloudWatch – Part 1

> AWS DevOps Playbook
>
> Volume 1 – Monitoring & Observability
>
> Chapter 24
>
> Amazon CloudWatch – Part 1 (Introduction, Metrics, Dashboards & Monitoring Fundamentals)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Amazon CloudWatch
- Learn Monitoring vs Observability
- Understand Metrics
- Learn Namespaces
- Understand Dimensions
- Learn Standard & Detailed Monitoring
- Understand CloudWatch Dashboards
- Monitor AWS Resources
- Answer Interview Questions

---

# 1. What is Amazon CloudWatch?

Amazon CloudWatch is a **monitoring and observability service** provided by AWS.

It helps monitor:

- AWS Resources
- Applications
- Infrastructure
- Logs
- Events
- Custom Metrics

Think of CloudWatch as the **health monitoring system** of your AWS infrastructure.

---

# 2. Why Do We Need Monitoring?

Imagine your production application suddenly becomes slow.

Without monitoring:

```
Users

↓

Application Slow

↓

Nobody Knows Why
```

With CloudWatch:

```
High CPU

↓

High Memory

↓

High Latency

↓

Alarm

↓

DevOps Team Notified
```

---

# 3. Real Production Scenario

Suppose you have:

```
10 EC2 Instances

↓

Application Load Balancer

↓

Amazon RDS

↓

S3
```

How do you know:

- CPU is high?
- Memory is full?
- Disk is almost full?
- Application is crashing?

Answer:

```
Amazon CloudWatch
```

---

# 4. Monitoring vs Observability

Many people think they are the same.

They are not.

### Monitoring

Answers:

> Is something wrong?

Example:

```
CPU = 95%
```

---

### Observability

Answers:

> Why is it wrong?

Example

```
CPU High

↓

Slow SQL

↓

Database Lock

↓

Application Delay
```

CloudWatch is a key part of AWS observability when combined with Logs, X-Ray, CloudTrail, and other services.

---

# 5. CloudWatch Architecture

```
AWS Resources

↓

CloudWatch Agent

↓

CloudWatch Metrics

↓

CloudWatch Dashboard

↓

CloudWatch Alarm

↓

SNS

↓

Email / Slack / Teams
```

---

# 6. AWS Services Integrated with CloudWatch

CloudWatch can monitor:

```
EC2

↓

RDS

↓

Lambda

↓

ALB

↓

NLB

↓

CloudFront

↓

S3

↓

DynamoDB

↓

ECS

↓

EKS

↓

API Gateway

↓

Custom Applications
```

Almost every AWS service publishes CloudWatch metrics.

---

# 7. CloudWatch Components

```
CloudWatch

│

├── Metrics

├── Dashboards

├── Alarms

├── Logs

├── Events

├── Contributor Insights

├── Application Insights

├── Synthetics

└── Evidently
```

In this chapter we focus on **Metrics** and **Dashboards**.

---

# 8. What is a Metric?

A metric is a numerical value measured over time.

Examples:

```
CPU Utilization

45%
```

```
Memory Usage

70%
```

```
Network In

200 MB
```

```
Disk Read

500 IOPS
```

---

# 9. Examples of AWS Metrics

### EC2

- CPUUtilization
- NetworkIn
- NetworkOut
- DiskReadOps
- DiskWriteOps
- StatusCheckFailed

---

### RDS

- CPUUtilization
- DatabaseConnections
- FreeStorageSpace
- ReadLatency
- WriteLatency

---

### ALB

- RequestCount
- TargetResponseTime
- HTTPCode_ELB_5XX
- HealthyHostCount

---

### Lambda

- Invocations
- Errors
- Duration
- Throttles

---

# 10. Metric Lifecycle

```
AWS Resource

↓

Collect Metric

↓

Store in CloudWatch

↓

Display on Dashboard

↓

Alarm

↓

Notification
```

---

# 11. Metric Resolution

CloudWatch supports multiple resolutions.

### Standard Resolution

```
1 Minute
```

Suitable for most workloads.

---

### High Resolution

```
1 Second
```

Useful for:

- Trading systems
- Real-time applications
- Low-latency workloads

---

# 12. Namespace

Metrics are grouped using namespaces.

Examples:

```
AWS/EC2

AWS/RDS

AWS/Lambda

AWS/S3

AWS/ApplicationELB
```

Custom applications use:

```
Custom Namespace

Example:

Company/Application
```

---

# 13. Dimensions

Dimensions identify a specific resource.

Example

Namespace

```
AWS/EC2
```

Metric

```
CPUUtilization
```

Dimension

```
InstanceId=i-123456789
```

This tells CloudWatch **which EC2 instance** the metric belongs to.

---

# 14. Example of Dimensions

```
AWS/EC2

↓

CPUUtilization

↓

Instance ID

↓

i-0abc1234
```

Without dimensions, CloudWatch cannot distinguish between multiple resources.

---

# 15. CloudWatch Dashboard

A Dashboard provides a graphical view of metrics.

Example

```
Dashboard

↓

CPU

↓

Memory

↓

Network

↓

Disk

↓

Errors
```

Everything is visible from a single screen.

---

# 16. Production Dashboard Example

```
Production Dashboard

-----------------------------------

EC2 CPU

RDS CPU

ALB Requests

Lambda Errors

Disk Usage

Network Traffic

Application Errors
```

Operations teams monitor this dashboard throughout the day.

---

# 17. Benefits of Dashboards

✔ Centralized Monitoring

✔ Easy Visualization

✔ Multiple Services

✔ Historical Trends

✔ Faster Troubleshooting

---

# 18. CloudWatch Data Retention

CloudWatch retains metrics for different durations depending on resolution.

Typical retention:

| Resolution | Retention |
|------------|-----------|
| 1 Second (High Resolution) | Short-term |
| 1 Minute | Weeks |
| 5 Minute | Months |
| 1 Hour | Long-term |

AWS automatically aggregates older data into coarser intervals.

---

# 19. Standard Monitoring vs Detailed Monitoring

### Standard Monitoring

```
5 Minute Metrics
```

Lower cost.

Suitable for development workloads.

---

### Detailed Monitoring

```
1 Minute Metrics
```

Faster visibility.

Recommended for production.

---

# 20. EC2 Monitoring Example

Without Detailed Monitoring

```
CPU Updated

Every

5 Minutes
```

With Detailed Monitoring

```
CPU Updated

Every

1 Minute
```

Faster detection of issues.

---

# 21. How Metrics Reach CloudWatch

```
EC2

↓

AWS Monitoring Service

↓

CloudWatch

↓

Dashboard
```

For operating system metrics like **memory** and **disk usage**, you generally need the **CloudWatch Agent** because EC2 publishes only a limited set of metrics by default.

---

# 22. CloudWatch Agent

The CloudWatch Agent collects:

- Memory Usage
- Disk Usage
- Swap Usage
- Running Processes
- Custom Logs

Architecture

```
Linux Server

↓

CloudWatch Agent

↓

CloudWatch
```

---

# 23. Native Metrics vs Agent Metrics

| Native AWS Metrics | CloudWatch Agent Metrics |
|--------------------|--------------------------|
| CPU | Memory |
| Network | Disk Usage |
| Status Checks | Processes |
| EBS Metrics | Swap |
| ELB Metrics | Custom OS Metrics |

---

# 24. Real DevOps Example

Suppose:

```
CPU

20%
```

Application is still slow.

CloudWatch Agent shows:

```
Memory

99%
```

Now the real problem is visible.

---

# 25. Best Practices

✔ Enable Detailed Monitoring for production.

✔ Install CloudWatch Agent on EC2.

✔ Organize dashboards by environment.

✔ Use meaningful dashboard names.

✔ Monitor trends, not just current values.

---

# 26. Common Mistakes

❌ Monitoring only CPU.

❌ Ignoring memory usage.

❌ No dashboards.

❌ No historical analysis.

❌ Not enabling Detailed Monitoring for production.

---

# 27. Interview Questions

## Question 1

What is Amazon CloudWatch?

### Answer

Amazon CloudWatch is a monitoring and observability service that collects metrics, logs, and events from AWS resources and applications, enabling visualization, alerting, and operational insights.

---

## Question 2

What is a metric?

### Answer

A metric is a numerical measurement collected over time, such as CPU utilization, network traffic, or database connections.

---

## Question 3

What is a namespace in CloudWatch?

### Answer

A namespace is a logical container used to organize metrics. Examples include `AWS/EC2`, `AWS/RDS`, and `AWS/Lambda`.

---

## Question 4

What is a dimension?

### Answer

A dimension is a name-value pair that uniquely identifies a metric, such as an EC2 Instance ID or Load Balancer name.

---

## Question 5

What is the difference between Standard Monitoring and Detailed Monitoring?

### Answer

Standard Monitoring publishes metrics every **5 minutes**, while Detailed Monitoring publishes metrics every **1 minute**, allowing faster detection of operational issues.

---

## Question 6

Why is the CloudWatch Agent required?

### Answer

The CloudWatch Agent collects operating system metrics such as memory usage, disk utilization, swap usage, and custom logs, which are not available by default.

---

# 28. Hands-on Labs

## Lab 1

Open the CloudWatch console and explore metrics for:

- EC2
- RDS
- Lambda
- ALB

---

## Lab 2

Create a dashboard displaying:

- EC2 CPU
- Network In
- Network Out

---

## Lab 3

Enable Detailed Monitoring on an EC2 instance and compare it with Standard Monitoring.

---

## Lab 4

Install the CloudWatch Agent on an EC2 instance and publish:

- Memory Usage
- Disk Usage

---

# 29. One-Page Revision

```
Amazon CloudWatch

↓

Metrics

↓

Namespaces

↓

Dimensions

↓

Dashboards

↓

Standard Monitoring

↓

Detailed Monitoring

↓

CloudWatch Agent

↓

Visualization

↓

Monitoring
```

Remember:

- Metrics are numerical values collected over time.
- Namespaces organize metrics.
- Dimensions identify specific resources.
- Dashboards visualize infrastructure health.
- Detailed Monitoring provides 1-minute metrics.
- CloudWatch Agent collects OS-level metrics.

---

# Think Like a DevOps Engineer

CloudWatch is not just a dashboard—it's your first line of defense against production issues.

A mature monitoring strategy should include:

1. Infrastructure metrics (CPU, Network, Disk).
2. Operating system metrics (Memory, Swap, Disk).
3. Application metrics (Request Count, Errors, Latency).
4. Centralized dashboards for operations teams.
5. Historical data analysis to identify trends before they become outages.

Monitoring tells you **what is happening**. In the next chapters, you'll learn how CloudWatch **Logs**, **Alarms**, and **Events** help you understand **why** it is happening and automate responses.

# End of Part 1
