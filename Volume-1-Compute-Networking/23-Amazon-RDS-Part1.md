# Amazon RDS

> AWS DevOps Playbook
>
> Volume 1 – Databases
>
> Chapter 23
>
> Amazon RDS – Part 1 (Introduction & Core Concepts)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand what Amazon RDS is
- Understand why RDS exists
- Compare Self-Managed Database vs Amazon RDS
- Understand supported database engines
- Learn RDS architecture
- Understand DB Instances
- Learn storage, compute and networking
- Understand RDS components
- Understand AWS Shared Responsibility Model
- Compare RDS with EC2 databases
- Design a production RDS architecture
- Answer common interview questions

---

# 1. What is Amazon RDS?

Amazon RDS stands for:

> **Amazon Relational Database Service**

It is a **fully managed relational database service** provided by AWS.

Instead of installing a database yourself on an EC2 instance, AWS manages the database infrastructure for you.

Example:

```
Traditional

Buy Server

↓

Install OS

↓

Install Database

↓

Configure Backup

↓

Configure Monitoring

↓

Maintain Database

-------------------------

AWS

Create RDS

↓

Database Ready
```

---

# 2. What is a Relational Database?

A relational database stores data in the form of:

- Tables
- Rows
- Columns

Example

Customer Table

| CustomerID | Name | City |
|------------|------|------|
|101|Rahul|Mumbai|
|102|Amit|Delhi|
|103|John|London|

Relationships between tables are created using:

- Primary Keys
- Foreign Keys

---

# 3. Why Amazon RDS?

Imagine you install MySQL on EC2.

You are responsible for:

- OS installation
- Database installation
- Security patches
- Database patches
- Backups
- Monitoring
- Failover
- Storage management
- Recovery
- High Availability

This takes significant operational effort.

Amazon RDS automates most of these tasks.

---

# 4. Problems with Self-Managed Databases

```
EC2

↓

Linux

↓

Install MySQL

↓

Patch OS

↓

Patch DB

↓

Configure Backup

↓

Monitor

↓

Scale

↓

Recover
```

Every task is your responsibility.

---

# 5. Benefits of Amazon RDS

Amazon RDS automates:

✔ Database installation

✔ Software patching

✔ Automatic backups

✔ Monitoring

✔ High Availability

✔ Storage scaling

✔ Failure detection

✔ Recovery

✔ Maintenance windows

---

# 6. Supported Database Engines

Amazon RDS supports multiple database engines.

| Database | Type |
|-----------|------|
| MySQL | Open Source |
| PostgreSQL | Open Source |
| MariaDB | Open Source |
| Oracle Database | Commercial |
| Microsoft SQL Server | Commercial |
| Amazon Aurora | AWS Managed Database |

Each engine has different licensing, performance characteristics, and features.

---

# 7. Amazon Aurora

Aurora is AWS's cloud-native relational database.

Compatible with:

- MySQL
- PostgreSQL

Benefits:

- Higher performance
- Faster failover
- Distributed storage
- Better scalability

Aurora is covered in a separate chapter.

---

# 8. Amazon RDS Architecture

```
Application

↓

Amazon RDS

↓

Database Engine

↓

Storage

↓

Automatic Backup
```

AWS manages:

- Hardware
- Operating System
- Database software
- Storage infrastructure

---

# 9. Components of Amazon RDS

Major components include:

```
RDS

│

├── DB Instance

├── Storage

├── Database Engine

├── Security Group

├── Parameter Group

├── Option Group

├── Monitoring

├── Backup

└── Snapshot
```

---

# 10. What is a DB Instance?

A **DB Instance** is the compute resource that runs your database.

It includes:

- CPU
- Memory
- Network
- Database Engine

Example

```
DB Instance

↓

MySQL

↓

8 vCPU

↓

32 GB RAM
```

Think of it as a managed virtual server dedicated to your database.

---

# 11. DB Instance Classes

Examples:

```
db.t3.micro

db.t3.small

db.t3.medium

db.m6i.large

db.r6g.large
```

General guideline:

- **t** family → Development / Testing
- **m** family → Balanced workloads
- **r** family → Memory-intensive databases

---

# 12. Storage in Amazon RDS

RDS supports multiple storage options.

```
Storage

│

├── General Purpose SSD (gp3/gp2)

├── Provisioned IOPS SSD

└── Magnetic (legacy, limited use)
```

We'll study these in Part 2.

---

# 13. Networking

Every RDS instance runs inside a VPC.

```
VPC

↓

Subnet

↓

DB Subnet Group

↓

RDS Instance
```

Important concepts:

- Private Subnets
- Security Groups
- Route Tables
- DB Subnet Groups

---

# 14. DB Subnet Group

A DB Subnet Group is a collection of subnets that Amazon RDS can use.

Example:

```
Subnet A

+

Subnet B

↓

DB Subnet Group

↓

Amazon RDS
```

For Multi-AZ deployments, the subnet group should include subnets in different Availability Zones.

---

# 15. Security Groups

Security Groups control:

```
Who Can Connect

↓

Port 3306

↓

MySQL
```

Example

Allow:

```
Application Server

↓

MySQL

↓

3306
```

Deny:

```
Internet

↓

Blocked
```

---

# 16. Endpoint

Unlike EC2,

you do not connect using an IP address.

You connect using a DNS endpoint.

Example:

```
mydb.abcdefg.us-east-1.rds.amazonaws.com
```

Applications always connect using the endpoint.

---

# 17. Ports

Common database ports:

| Database | Port |
|----------|------|
| MySQL | 3306 |
| PostgreSQL | 5432 |
| Oracle | 1521 |
| SQL Server | 1433 |
| MariaDB | 3306 |

Ensure Security Groups allow only trusted sources.

---

# 18. Shared Responsibility Model

AWS manages:

- Hardware
- Networking
- Storage hardware
- Database software installation
- Automated backups infrastructure
- Patching (depending on configuration)

Customer manages:

- Database schema
- Users
- Passwords
- Queries
- IAM permissions
- Security Groups
- Application-level security
- Data

---

# 19. RDS vs Database on EC2

| Feature | RDS | EC2 Database |
|----------|-----|--------------|
| OS Management | AWS | Customer |
| DB Installation | AWS | Customer |
| Backups | Automated | Manual |
| Monitoring | Built-in | Manual |
| Failover | Supported | Manual setup |
| Patching | Managed | Customer |
| Storage Scaling | Supported | Manual |
| High Availability | Multi-AZ | Customer designs it |

---

# 20. Typical Production Architecture

```
Users

↓

Application Load Balancer

↓

Application Servers

↓

Amazon RDS

↓

Automated Backup

↓

CloudWatch
```

The database is typically placed in **private subnets** and is not directly accessible from the Internet.

---

# 21. Real-World Example

An e-commerce application:

```
Users

↓

Website

↓

Spring Boot

↓

Amazon RDS MySQL

↓

Product Database
```

The application stores:

- Customers
- Orders
- Products
- Payments

in Amazon RDS.

---

# 22. Best Practices

✔ Place RDS in private subnets.

✔ Allow access only from application Security Groups.

✔ Enable automatic backups.

✔ Enable deletion protection for production databases.

✔ Use Multi-AZ for production workloads.

✔ Monitor storage and CPU usage.

✔ Enable encryption where required.

✔ Apply least-privilege access.

---

# 23. Common Mistakes

❌ Exposing RDS to the public Internet.

❌ Using the master user for applications.

❌ Disabling backups.

❌ No monitoring.

❌ No Multi-AZ for production.

❌ Weak passwords.

❌ Opening database ports to `0.0.0.0/0`.

---

# 24. Production Scenario

A banking application requires:

- High Availability
- Automated backups
- Monitoring
- Secure access

Architecture:

```
Application

↓

Private Security Group

↓

Amazon RDS

↓

Private Subnet

↓

Automated Backup

↓

CloudWatch

↓

KMS Encryption
```

---

# 25. Interview Questions

## Question 1

What is Amazon RDS?

### Answer

Amazon RDS is a fully managed relational database service that automates tasks such as provisioning, backups, patching, monitoring, and high availability.

---

## Question 2

Which database engines are supported by Amazon RDS?

### Answer

- MySQL
- PostgreSQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Amazon Aurora

---

## Question 3

What is a DB Instance?

### Answer

A DB Instance is the compute resource that runs the database engine, including CPU, memory, storage, and networking.

---

## Question 4

Can Amazon RDS run inside a VPC?

### Answer

Yes.

Every Amazon RDS instance is launched inside an Amazon VPC.

---

## Question 5

What is the difference between RDS and installing MySQL on EC2?

### Answer

With RDS, AWS manages infrastructure, backups, patching, and maintenance. On EC2, you manage the entire database stack yourself.

---

## Question 6

What is an RDS endpoint?

### Answer

An endpoint is the DNS hostname that applications use to connect to the database.

---

## Question 7

Should an RDS database be publicly accessible?

### Answer

For production workloads, no. It should typically reside in private subnets and be accessible only from trusted application servers.

---

# 26. Hands-on Labs

## Lab 1

Create a MySQL RDS instance.

---

## Lab 2

Create a PostgreSQL RDS instance.

---

## Lab 3

Connect to the RDS instance using a database client.

---

## Lab 4

Create tables and insert sample data.

---

## Lab 5

Delete the database after taking a manual snapshot.

---

# 27. One-Page Revision

```
Amazon RDS

↓

Managed Database

↓

MySQL

PostgreSQL

MariaDB

Oracle

SQL Server

Aurora

↓

DB Instance

↓

Storage

↓

Security Group

↓

Endpoint

↓

Backup

↓

Monitoring
```

Remember:

- RDS = Managed relational database service.
- Runs inside a VPC.
- Uses DB Instances.
- Connect using a DNS endpoint.
- Supports multiple database engines.
- AWS manages infrastructure; you manage your data and access.

---

# Think Like a Production Engineer

A production database is not just about storing data—it must be **secure, highly available, recoverable, and monitored**.

When designing Amazon RDS:

1. Deploy it in private subnets.
2. Restrict access using Security Groups.
3. Enable automated backups.
4. Use Multi-AZ for high availability.
5. Monitor performance with CloudWatch.
6. Encrypt sensitive data.
7. Regularly test backup and recovery procedures.

A production-ready Amazon RDS deployment is **secure, resilient, scalable, and operationally efficient**.

# End of Part 1