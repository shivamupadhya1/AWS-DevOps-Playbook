# Amazon RDS

> AWS DevOps Playbook
>
> Volume 1 – Databases
>
> Chapter 23
>
> Amazon RDS – Part 7 (DevOps, Terraform, Kubernetes, Monitoring & Production Operations)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Deploy Amazon RDS using Terraform
- Deploy RDS using CloudFormation
- Integrate RDS with Jenkins
- Use Amazon RDS with ECS
- Use Amazon RDS with Kubernetes (EKS)
- Connect Lambda with RDS
- Monitor production databases
- Understand maintenance activities
- Understand DevOps best practices
- Answer real production interview questions

---

# 187. Amazon RDS in DevOps

Amazon RDS is rarely created manually in enterprise environments.

Instead, infrastructure is managed using **Infrastructure as Code (IaC)**.

Typical workflow:

```
GitHub

↓

Terraform

↓

AWS

↓

Amazon RDS
```

Benefits:

- Version Control
- Automation
- Repeatability
- Easy Rollback
- Auditing

---

# 188. RDS using Terraform

Typical resources used:

```
Terraform

│

├── aws_db_instance

├── aws_db_subnet_group

├── aws_security_group

├── aws_kms_key

├── aws_secretsmanager_secret

└── aws_cloudwatch_metric_alarm
```

A complete production deployment generally provisions all of these together.

---

# 189. Terraform Architecture

```
Developer

↓

Git Push

↓

GitHub

↓

Jenkins

↓

Terraform Apply

↓

Amazon RDS
```

Infrastructure changes become part of the CI/CD pipeline.

---

# 190. Important Terraform Parameters

Typical parameters include:

```
engine

engine_version

instance_class

allocated_storage

storage_type

multi_az

backup_retention_period

storage_encrypted

kms_key_id

db_subnet_group_name

vpc_security_group_ids

deletion_protection
```

Parameterize these values using variables.

---

# 191. RDS using CloudFormation

CloudFormation also supports RDS provisioning.

```
YAML

↓

CloudFormation

↓

Amazon RDS
```

Suitable for organizations standardized on CloudFormation.

---

# 192. CI/CD Pipeline

Typical deployment pipeline:

```
Developer

↓

GitHub

↓

Jenkins

↓

Terraform

↓

Amazon RDS

↓

Application Deployment
```

Database infrastructure is created before application deployment.

---

# 193. Should CI/CD Create Production Databases?

Usually:

```
NO
```

Production databases are generally provisioned separately and modified only through controlled change-management processes.

Application deployments should not recreate production databases.

---

# 194. Schema Deployment

Infrastructure and schema are different.

```
Terraform

↓

Database

↓

Flyway / Liquibase

↓

Tables

Indexes

Procedures
```

Terraform creates infrastructure.

Migration tools manage database schema.

---

# 195. Flyway

Flyway is a database migration tool.

Example

```
V1__create_tables.sql

↓

V2__add_index.sql

↓

V3__add_column.sql
```

Every deployment applies pending migrations automatically.

---

# 196. Liquibase

Liquibase is another migration framework.

Supports:

- SQL
- XML
- YAML
- JSON

Useful for enterprise environments.

---

# 197. Amazon RDS with ECS

Architecture

```
Internet

↓

ALB

↓

Amazon ECS

↓

Amazon RDS
```

Best Practices

- Private Subnets
- Security Groups
- Secrets Manager
- IAM Task Roles

---

# 198. ECS Connectivity

```
ECS Task

↓

Security Group

↓

RDS Security Group

↓

3306
```

Never expose the database publicly.

---

# 199. Amazon RDS with Kubernetes (EKS)

Architecture

```
Users

↓

Ingress

↓

Kubernetes Service

↓

Pods

↓

Amazon RDS
```

Pods remain stateless.

Persistent business data stays in RDS.

---

# 200. Kubernetes Best Practices

```
Application

↓

Secrets

↓

ConfigMap

↓

Amazon RDS
```

Store:

- Endpoint
- Username
- Password

securely using Kubernetes Secrets (or integrate with AWS Secrets Manager).

---

# 201. Amazon RDS with Lambda

Serverless applications often access RDS.

Architecture

```
API Gateway

↓

Lambda

↓

Amazon RDS
```

---

# 202. Lambda Connection Challenges

Every Lambda invocation may create a new database connection.

```
1000 Requests

↓

1000 Lambda Invocations

↓

1000 Database Connections
```

This can exhaust the database connection limit.

---

# 203. Amazon RDS Proxy

Amazon RDS Proxy helps manage database connections.

Architecture

```
Lambda

↓

Amazon RDS Proxy

↓

Amazon RDS
```

Benefits

✔ Connection Pooling

✔ Faster Connections

✔ Better Scalability

✔ Improved Availability

---

# 204. Secrets Manager Integration

Instead of storing credentials inside code:

```
Lambda

↓

Secrets Manager

↓

Amazon RDS
```

Credentials can be rotated automatically.

---

# 205. Production Monitoring

Monitor continuously:

```
CPU

Memory

Connections

Storage

Latency

Replica Lag

Failover Events
```

Monitoring should be proactive, not reactive.

---

# 206. CloudWatch Dashboard

Example dashboard

```
CloudWatch

↓

CPU

↓

Connections

↓

Storage

↓

Latency

↓

Replica Lag
```

Use dashboards for operational visibility.

---

# 207. CloudWatch Alarms

Example alarms

```
CPU > 80%

↓

SNS

↓

Email

↓

DevOps Team
```

Other useful alarms:

- Free Storage
- Replica Lag
- Freeable Memory
- High Read Latency

---

# 208. Logging

Enable logs such as:

- Error Logs
- General Logs
- Slow Query Logs
- Audit Logs (engine-dependent)

Export supported logs to CloudWatch Logs.

---

# 209. Maintenance Activities

Routine maintenance includes:

- Minor version upgrades
- Parameter review
- Index optimization
- Statistics updates
- Backup verification
- Restore testing

---

# 210. Capacity Planning

Track trends instead of waiting for failures.

```
CPU

↓

Monthly Growth

↓

Scale Before

↓

Performance Drops
```

Forecast future capacity needs.

---

# 211. Production Health Checklist

```
✔ Multi-AZ Enabled

✔ Automated Backups Enabled

✔ CloudWatch Monitoring

✔ Performance Insights

✔ Encryption Enabled

✔ Secrets Manager

✔ CloudTrail Enabled

✔ Read Replicas (if required)

✔ Parameter Group Reviewed

✔ Backup Tested

✔ Restore Tested

✔ Deletion Protection Enabled
```

---

# 212. Cost Optimization

Ways to reduce cost:

✔ Stop development databases when possible.

✔ Delete unused snapshots.

✔ Choose the correct instance size.

✔ Use Reserved DB Instances for predictable workloads.

✔ Avoid overprovisioning storage.

✔ Remove unused Read Replicas.

---

# 213. Common Production Issues

```
High CPU

↓

Slow Queries

↓

Missing Indexes

↓

Storage Full

↓

Too Many Connections

↓

Replica Lag

↓

Application Timeouts
```

A systematic troubleshooting process is essential.

---

# 214. Real Production Architecture

```
Users

↓

CloudFront

↓

ALB

↓

ECS / EKS

↓

Amazon RDS Proxy

↓

Primary RDS

↓

Multi-AZ

↓

Read Replicas

↓

AWS Backup

↓

CloudWatch

↓

CloudTrail

↓

Secrets Manager
```

---

# 215. Production Deployment Checklist

Before Go-Live

```
✔ Private Subnets

✔ Security Groups

✔ SSL Enabled

✔ Encryption Enabled

✔ Backups Configured

✔ Monitoring Enabled

✔ CloudWatch Alarms

✔ Performance Insights

✔ Secrets Manager

✔ Parameter Group Validated

✔ Maintenance Window Selected

✔ Restore Tested
```

---

# 216. Interview Questions

## Question 49

How do you deploy Amazon RDS using Infrastructure as Code?

### Answer

Using tools such as Terraform or AWS CloudFormation to provision RDS instances, subnet groups, security groups, monitoring, encryption, and backups in a repeatable manner.

---

## Question 50

Why should Flyway or Liquibase be used with Amazon RDS?

### Answer

Terraform provisions infrastructure, while Flyway and Liquibase manage database schema versions, ensuring controlled and repeatable schema changes.

---

## Question 51

How does Kubernetes connect to Amazon RDS?

### Answer

Application Pods connect to the RDS endpoint over the VPC network using Security Groups. Credentials are stored securely using Kubernetes Secrets or AWS Secrets Manager.

---

## Question 52

Why is Amazon RDS Proxy used with AWS Lambda?

### Answer

RDS Proxy pools and manages database connections, preventing connection exhaustion caused by rapidly scaling Lambda functions.

---

## Question 53

Which monitoring tools should be enabled?

### Answer

- CloudWatch
- Performance Insights
- Enhanced Monitoring
- CloudTrail
- CloudWatch Logs

---

## Question 54

Should database credentials be stored in application.properties?

### Answer

No.

Production environments should use AWS Secrets Manager or another secure secret-management solution.

---

## Question 55

How do you monitor Amazon RDS in production?

### Answer

Monitor CPU utilization, memory, storage, latency, database connections, replica lag, backups, failover events, and query performance using CloudWatch and Performance Insights.

---

## Question 56

How do you perform schema changes in production?

### Answer

Use controlled database migration tools such as Flyway or Liquibase through the CI/CD pipeline instead of manually executing SQL on production.

---

# 217. Hands-on Labs

## Lab 35

Provision Amazon RDS using Terraform.

---

## Lab 36

Deploy a Spring Boot application to ECS and connect it to Amazon RDS.

---

## Lab 37

Deploy an application on Amazon EKS using RDS as the backend database.

---

## Lab 38

Create an AWS Lambda function that accesses Amazon RDS using Amazon RDS Proxy.

---

## Lab 39

Configure CloudWatch dashboards and alarms for Amazon RDS.

---

## Lab 40

Integrate Flyway into a CI/CD pipeline and perform an automated schema migration.

---

# 218. One-Page Revision

```
Amazon RDS

↓

Terraform

↓

CloudFormation

↓

Flyway

↓

Liquibase

↓

ECS

↓

EKS

↓

Lambda

↓

RDS Proxy

↓

Secrets Manager

↓

CloudWatch

↓

Performance Insights

↓

CloudTrail

↓

AWS Backup
```

Remember:

- Terraform provisions infrastructure.
- Flyway/Liquibase manage schema.
- ECS and EKS connect through private networking.
- Lambda should use RDS Proxy.
- Secrets belong in Secrets Manager.
- Monitor continuously with CloudWatch.
- Test backup and restore regularly.

---

# Think Like a Senior DevOps Engineer

Managing Amazon RDS in production is more than creating a database.

A mature DevOps workflow includes:

1. Provision infrastructure using **Terraform** or **CloudFormation**.
2. Manage schema changes with **Flyway** or **Liquibase**.
3. Secure credentials using **Secrets Manager**.
4. Use **RDS Proxy** for highly concurrent serverless workloads.
5. Monitor health with **CloudWatch**, **Performance Insights**, and **CloudTrail**.
6. Validate backups by performing restore tests.
7. Continuously optimize performance, security, availability, and cost.

The goal is to build a database platform that is **automated, secure, observable, resilient, and easy to operate at scale**.

# End of Part 7