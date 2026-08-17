# Amazon CloudWatch – Part 3

> AWS DevOps Playbook
>
> Volume 1 – Monitoring & Observability
>
> Chapter 24
>
> Amazon CloudWatch – Part 3 (CloudWatch Alarms, Logs Insights, Metric Filters, Events & Production Monitoring)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand CloudWatch Alarms
- Learn Alarm States
- Configure SNS Notifications
- Understand Metric Filters
- Learn CloudWatch Logs Insights
- Understand Composite Alarms
- Learn CloudWatch Events / EventBridge
- Design Production Monitoring
- Answer Interview Questions

---

# 62. What is a CloudWatch Alarm?

A CloudWatch Alarm continuously monitors a metric and performs an action when a threshold is reached.

Example

```
CPU > 80%

↓

Alarm

↓

Send Email
```

---

# 63. Why Do We Need Alarms?

Monitoring alone is not enough.

Without alarms

```
CPU

95%

↓

Nobody Notices
```

With alarms

```
CPU

95%

↓

CloudWatch Alarm

↓

SNS

↓

Email / SMS / Teams

↓

DevOps Team
```

---

# 64. Alarm Workflow

```
AWS Resource

↓

Metric

↓

CloudWatch

↓

Alarm

↓

SNS

↓

Notification
```

---

# 65. Alarm States

Every CloudWatch Alarm has three possible states.

```
OK

↓

Everything Healthy
```

---

```
ALARM

↓

Threshold Breached
```

---

```
INSUFFICIENT_DATA

↓

CloudWatch Does Not Have Enough Data
```

Example:

- Newly created EC2 instance
- Metric not yet published
- Temporary metric interruption

---

# 66. Example Alarm

Condition

```
CPUUtilization

>

80%

for

5 Minutes
```

Action

```
Send Email
```

---

# 67. Real Production Example

```
EC2

↓

CPU

92%

↓

Alarm

↓

SNS

↓

DevOps Engineer

↓

Scale EC2
```

---

# 68. Alarm Actions

CloudWatch can trigger:

- SNS Notification
- EC2 Recovery
- Auto Scaling Action
- Lambda Function
- Systems Manager Automation

Example

```
High CPU

↓

Auto Scaling

↓

Launch New EC2
```

---

# 69. SNS Integration

Architecture

```
CloudWatch Alarm

↓

SNS Topic

↓

Subscribers

↓

Email

SMS

Lambda

HTTPS Endpoint
```

SNS distributes notifications to multiple subscribers.

---

# 70. Example Production Alarm

```
Application

↓

ALB

↓

HTTP 5XX

>

10

↓

Alarm

↓

SNS

↓

Operations Team
```

---

# 71. Common Alarm Metrics

### EC2

- CPUUtilization
- StatusCheckFailed
- NetworkIn
- NetworkOut

---

### RDS

- CPUUtilization
- DatabaseConnections
- FreeStorageSpace
- ReplicaLag

---

### ALB

- RequestCount
- HTTPCode_ELB_5XX_Count
- TargetResponseTime
- HealthyHostCount

---

### Lambda

- Errors
- Duration
- Throttles

---

# 72. Composite Alarms

A Composite Alarm combines multiple alarms.

Example

Instead of:

```
CPU Alarm

Memory Alarm

Disk Alarm
```

Use

```
Composite Alarm

↓

CPU

AND

Memory

↓

Single Alert
```

Benefits

- Fewer alerts
- Reduced alert fatigue
- Better incident quality

---

# 73. Alarm Evaluation

CloudWatch evaluates:

```
Metric

↓

Threshold

↓

Evaluation Period

↓

Alarm State
```

Example

```
CPU > 80%

5 Consecutive Minutes
```

---

# 74. Metric Filters

Metric Filters convert log entries into CloudWatch Metrics.

Example Log

```
ERROR

Database Timeout
```

Metric Filter

```
ERROR

↓

Counter +1
```

Now CloudWatch can create an alarm from log data.

---

# 75. Real Metric Filter Example

Spring Boot Log

```
ERROR

Payment Failed
```

Metric Filter

```
Pattern

ERROR
```

Generated Metric

```
ApplicationErrors
```

Alarm

```
ApplicationErrors

>

5

↓

Notify Team
```

---

# 76. CloudWatch Logs Insights

Logs Insights is used to search and analyze logs.

Instead of manually opening thousands of log files,

you can query them.

Example

```
Millions of Logs

↓

Logs Insights

↓

Search

↓

Results
```

---

# 77. Example Logs Insights Query

Find all ERROR logs

```
fields @timestamp,@message

| filter @message like /ERROR/

| sort @timestamp desc
```

---

# 78. Another Example

Find slow requests

```
fields @timestamp,@message

| filter @message like /timeout/
```

---

# 79. Why Logs Insights?

Suppose

```
100 EC2 Servers

↓

500 GB Logs
```

Instead of checking manually,

run a single query.

Huge time saver.

---

# 80. CloudWatch Dashboards + Logs

Production Dashboard

```
CPU

↓

Alarm

↓

Logs Insights

↓

Root Cause
```

Metrics identify **what** happened.

Logs explain **why**.

---

# 81. CloudWatch Events

CloudWatch Events has evolved into **Amazon EventBridge**, but many engineers still use the older name.

Example

```
EC2 Stopped

↓

Event

↓

Lambda

↓

Restart Instance
```

---

# 82. Example Events

- EC2 State Change
- RDS Failover
- Auto Scaling Event
- ECS Task Stopped
- CodePipeline State Change
- Backup Completed

---

# 83. Production Architecture

```
AWS Services

↓

CloudWatch Metrics

↓

CloudWatch Alarm

↓

SNS

↓

Email

↓

DevOps Team

↓

Logs Insights

↓

Root Cause Analysis
```

---

# 84. Monitoring Strategy

Monitor

```
Infrastructure

↓

Application

↓

Database

↓

Network

↓

Security
```

Do not monitor only EC2.

---

# 85. Golden Signals

Google's SRE model recommends monitoring:

```
Latency

Traffic

Errors

Saturation
```

AWS teams commonly use these four signals in production monitoring.

---

# 86. Production Dashboard Example

```
Dashboard

--------------------------------

EC2 CPU

EC2 Memory

Disk Usage

ALB Requests

HTTP 5XX

Lambda Errors

RDS CPU

Replica Lag

Network

Storage

Application Errors
```

---

# 87. Best Practices

✔ Create alarms for critical metrics.

✔ Avoid alert fatigue.

✔ Use Composite Alarms.

✔ Monitor business metrics.

✔ Use Logs Insights regularly.

✔ Configure SNS notifications.

✔ Test alarms periodically.

✔ Review alarm thresholds.

---

# 88. Common Mistakes

❌ Hundreds of unnecessary alarms.

❌ Alerting on every warning.

❌ No log analysis.

❌ Never testing alarms.

❌ No notification recipients.

❌ Ignoring alarm history.

---

# 89. Interview Questions

## Question 13

What is a CloudWatch Alarm?

### Answer

A CloudWatch Alarm monitors a metric against a defined threshold and performs configured actions such as sending notifications or triggering automation.

---

## Question 14

What are the three alarm states?

### Answer

- OK
- ALARM
- INSUFFICIENT_DATA

---

## Question 15

What is Amazon SNS used for with CloudWatch?

### Answer

SNS distributes alarm notifications to subscribers such as email addresses, SMS recipients, Lambda functions, or HTTPS endpoints.

---

## Question 16

What is a Metric Filter?

### Answer

A Metric Filter converts matching log patterns into CloudWatch Metrics, allowing alarms to be created from log data.

---

## Question 17

What is CloudWatch Logs Insights?

### Answer

CloudWatch Logs Insights is a query service used to search, analyze, and troubleshoot CloudWatch Logs efficiently.

---

## Question 18

What is a Composite Alarm?

### Answer

A Composite Alarm combines multiple underlying alarms into a single alarm, reducing duplicate notifications and alert fatigue.

---

## Question 19

How do you monitor application errors?

### Answer

Collect application logs in CloudWatch Logs, create Metric Filters for ERROR patterns, generate custom metrics, and create CloudWatch Alarms that notify the operations team.

---

## Question 20

What should every production CloudWatch dashboard include?

### Answer

- CPU
- Memory
- Disk Usage
- Network
- Application Errors
- Request Count
- Latency
- Database Metrics
- Load Balancer Metrics
- Alarm Status

---

# 90. Hands-on Labs

## Lab 9

Create a CloudWatch Alarm for:

```
CPU > 80%
```

---

## Lab 10

Create an SNS Topic and subscribe your email.

Attach the topic to the CPU alarm.

---

## Lab 11

Create a Metric Filter that counts:

```
ERROR
```

messages from application logs.

---

## Lab 12

Create a Logs Insights query to display all ERROR messages from the last 24 hours.

---

## Lab 13

Create a Composite Alarm using:

- CPU Alarm
- Memory Alarm

---

## Lab 14

Build a production dashboard displaying:

- EC2 CPU
- Memory
- ALB Requests
- Lambda Errors
- RDS CPU
- Application Error Count

---

# 91. One-Page Revision

```
CloudWatch

↓

Metrics

↓

Alarm

↓

SNS

↓

Notification

↓

Logs

↓

Metric Filters

↓

Logs Insights

↓

Composite Alarms

↓

Dashboard

↓

EventBridge
```

Remember:

- Metrics tell you **what happened**.
- Logs tell you **why it happened**.
- Alarms notify the operations team.
- SNS distributes notifications.
- Metric Filters convert logs into metrics.
- Logs Insights makes troubleshooting fast.
- Composite Alarms reduce unnecessary alerts.

---

# Think Like a Senior DevOps Engineer

A mature monitoring platform does more than collect metrics—it enables **rapid detection, investigation, and response**.

A production-ready CloudWatch implementation should:

1. Monitor infrastructure, applications, and databases.
2. Generate meaningful alarms with well-defined thresholds.
3. Route notifications through SNS to the appropriate teams.
4. Use Metric Filters to detect application failures from logs.
5. Investigate incidents quickly with Logs Insights.
6. Reduce noise using Composite Alarms.
7. Review dashboards and alarm effectiveness regularly.

The goal is not to create **more alarms**, but to create **the right alarms** that help engineers respond quickly and confidently.

# End of Amazon CloudWatch (3 Parts)