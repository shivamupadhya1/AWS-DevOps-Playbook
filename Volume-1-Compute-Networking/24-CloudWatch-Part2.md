# Amazon CloudWatch – Part 2

> AWS DevOps Playbook
>
> Volume 1 – Monitoring & Observability
>
> Chapter 24
>
> Amazon CloudWatch – Part 2 (CloudWatch Logs, Log Groups, Log Streams & CloudWatch Agent)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand CloudWatch Logs
- Learn Log Groups
- Learn Log Streams
- Understand Log Retention
- Install CloudWatch Agent
- Monitor Linux Servers
- Centralize Application Logs
- Understand Production Logging
- Answer Interview Questions

---

# 30. What are CloudWatch Logs?

Metrics tell us:

> Something is wrong.

Logs tell us:

> Why it is wrong.

Example

```
Metric

↓

CPU = 95%
```

This tells us there is a problem.

Now we check:

```
CloudWatch Logs

↓

OutOfMemoryError
```

Now we know the reason.

---

# 31. Why Logs Matter

Suppose your application crashes.

Without logs

```
Application Failed

↓

Unknown Reason
```

With logs

```
Application Failed

↓

Stack Trace

↓

Database Timeout

↓

Problem Identified
```

---

# 32. What Can Be Stored in CloudWatch Logs?

CloudWatch Logs can collect:

- Linux System Logs
- Application Logs
- Nginx Logs
- Apache Logs
- Spring Boot Logs
- Docker Logs
- ECS Logs
- Lambda Logs
- Custom Application Logs

---

# 33. CloudWatch Logs Architecture

```
EC2

↓

CloudWatch Agent

↓

CloudWatch Logs

↓

Log Group

↓

Log Stream
```

---

# 34. Components of CloudWatch Logs

```
CloudWatch Logs

│

├── Log Groups

├── Log Streams

├── Log Events

├── Metric Filters

├── Subscription Filters

└── Retention Policies
```

---

# 35. What is a Log Group?

A Log Group is a logical collection of related logs.

Example

```
Production

↓

Spring Boot Logs
```

Another example

```
/aws/lambda/payment-service
```

---

# 36. Real Examples of Log Groups

```
/aws/ec2/nginx

/application/prod

/application/dev

/aws/lambda/orders

/aws/ecs/backend

/aws/rds/mysql
```

Production systems usually separate log groups by:

- Environment
- Application
- Team

---

# 37. What is a Log Stream?

Inside every Log Group,

there are multiple Log Streams.

Example

```
Log Group

↓

Spring Boot

↓

Instance 1

↓

Instance 2

↓

Instance 3
```

Each instance writes to its own stream.

---

# 38. Log Hierarchy

```
CloudWatch Logs

↓

Log Group

↓

Log Stream

↓

Log Events
```

---

# 39. Example

```
Log Group

/application/prod

↓

Log Stream

i-0ab12345

↓

Log Events

INFO

ERROR

WARN
```

---

# 40. What is a Log Event?

A Log Event is a single log message.

Example

```
2026-08-17

INFO

Application Started
```

Another

```
ERROR

Database Connection Failed
```

---

# 41. Example Spring Boot Logs

```
INFO

Tomcat Started

----------------

INFO

Connected to Database

----------------

WARN

Slow Query

----------------

ERROR

Connection Timeout
```

---

# 42. Linux System Logs

The CloudWatch Agent can send:

Ubuntu

```
/var/log/syslog
```

Amazon Linux

```
/var/log/messages
```

Authentication Logs

```
/var/log/secure

or

/var/log/auth.log
```

---

# 43. Nginx Logs

Access Log

```
/var/log/nginx/access.log
```

Error Log

```
/var/log/nginx/error.log
```

Both can be shipped to CloudWatch.

---

# 44. Docker Logs

Containers produce logs continuously.

```
Docker Container

↓

stdout

↓

CloudWatch Logs
```

This is commonly used in ECS.

---

# 45. ECS Logging

```
Container

↓

awslogs Driver

↓

CloudWatch Logs
```

Each ECS Task writes logs automatically.

---

# 46. Lambda Logging

Every Lambda function automatically sends logs.

```
Lambda

↓

CloudWatch Logs

↓

Log Group
```

No agent installation is required.

---

# 47. CloudWatch Agent

The CloudWatch Agent is installed on:

- EC2
- On-premises Servers
- Virtual Machines

It collects:

- Logs
- Memory
- Disk
- Processes

---

# 48. CloudWatch Agent Architecture

```
Linux Server

↓

CloudWatch Agent

↓

CloudWatch Logs

↓

Dashboard

↓

CloudWatch Insights
```

---

# 49. Installing CloudWatch Agent

Typical Steps

```
Install Package

↓

IAM Role

↓

Configuration File

↓

Start Agent

↓

Verify Logs
```

---

# 50. IAM Permissions

The EC2 instance needs permission.

Example Policy

```
CloudWatchAgentServerPolicy
```

Without permission,

the agent cannot publish logs.

---

# 51. CloudWatch Agent Configuration

Configuration includes:

```
Log File

↓

Log Group

↓

Retention

↓

Region

↓

Instance ID
```

Example

```
/var/log/messages

↓

CloudWatch
```

---

# 52. Log Retention

By default,

logs may be retained indefinitely unless configured otherwise.

Typical retention periods:

```
7 Days

30 Days

90 Days

180 Days

1 Year

Forever
```

Choose based on business and compliance requirements.

---

# 53. Why Log Retention Matters

Without retention

```
Logs

↓

Forever

↓

Higher Cost
```

With retention

```
30 Days

↓

Automatic Deletion
```

Cost is reduced.

---

# 54. Production Logging Strategy

```
Application

↓

CloudWatch Logs

↓

30 Days

↓

Archive (if required)

↓

Delete
```

---

# 55. Centralized Logging

Instead of checking:

```
Server 1

Server 2

Server 3

Server 4
```

Use

```
CloudWatch Logs

↓

Single Location
```

Much easier for troubleshooting.

---

# 56. Real DevOps Example

Suppose

```
ALB

↓

5XX Errors
```

Go to:

```
CloudWatch Logs

↓

Application

↓

Stack Trace

↓

Root Cause
```

---

# 57. Best Practices

✔ Separate Log Groups by application.

✔ Configure retention.

✔ Do not store logs forever unless required.

✔ Centralize logs.

✔ Use IAM Roles instead of access keys.

✔ Monitor error logs regularly.

---

# 58. Common Mistakes

❌ No log retention.

❌ Everything in one Log Group.

❌ No IAM Role.

❌ Ignoring ERROR logs.

❌ Logging sensitive information such as passwords or API keys.

---

# 59. Interview Questions

## Question 7

What is CloudWatch Logs?

### Answer

CloudWatch Logs is a managed logging service used to collect, centralize, store, and analyze logs from AWS resources and applications.

---

## Question 8

What is a Log Group?

### Answer

A Log Group is a logical collection of related log streams, usually grouped by application, environment, or AWS service.

---

## Question 9

What is a Log Stream?

### Answer

A Log Stream is a sequence of log events from a single source, such as one EC2 instance, container, or Lambda invocation.

---

## Question 10

Why is the CloudWatch Agent required?

### Answer

The CloudWatch Agent collects operating system logs and custom application logs from EC2 or on-premises servers and sends them to CloudWatch.

---

## Question 11

Does Lambda require the CloudWatch Agent?

### Answer

No.

AWS Lambda automatically publishes logs to CloudWatch Logs.

---

## Question 12

Why should log retention be configured?

### Answer

Proper retention reduces storage costs while meeting operational and compliance requirements.

---

# 60. Hands-on Labs

## Lab 5

Create a Log Group named:

```
production-app
```

---

## Lab 6

Install the CloudWatch Agent on an EC2 instance.

---

## Lab 7

Send:

- `/var/log/messages`
- `/var/log/nginx/error.log`

to CloudWatch Logs.

---

## Lab 8

Configure a retention policy of **30 days** for an application Log Group.

---

# 61. One-Page Revision

```
CloudWatch Logs

↓

Log Groups

↓

Log Streams

↓

Log Events

↓

CloudWatch Agent

↓

Linux Logs

↓

Application Logs

↓

Centralized Logging

↓

Retention Policy
```

Remember:

- Logs explain **why** an issue occurred.
- Log Groups organize related logs.
- Log Streams represent individual log sources.
- CloudWatch Agent sends logs from EC2 and on-premises servers.
- Configure retention to control costs.
- Centralized logging simplifies troubleshooting.

---

# Think Like a DevOps Engineer

Monitoring without logs is incomplete.

A production-grade logging strategy should:

1. Centralize logs from all servers and applications.
2. Separate logs by application and environment.
3. Configure retention based on compliance needs.
4. Avoid logging secrets or sensitive information.
5. Make logs searchable so incidents can be investigated quickly.

In the next chapter, you'll learn **CloudWatch Alarms, Metric Filters, Logs Insights, and Notifications**, which transform logs and metrics into automated operational responses.

# End of Part 2