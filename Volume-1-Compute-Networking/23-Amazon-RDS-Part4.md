# Amazon RDS

> AWS DevOps Playbook
>
> Volume 1 – Databases
>
> Chapter 23
>
> Amazon RDS – Part 4 (Security, Encryption, Monitoring & Database Administration)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand RDS Security
- Learn Security Groups
- Understand IAM Database Authentication
- Learn Encryption
- Understand KMS Integration
- Learn SSL/TLS Connections
- Understand Parameter Groups
- Learn Option Groups
- Understand Enhanced Monitoring
- Learn Performance Insights
- Understand CloudWatch Integration
- Answer Production Interview Questions

---

# 93. Security in Amazon RDS

Database security is one of the most important aspects of production systems.

A production database should never be exposed directly to the Internet.

Security Layers

```
Internet

↓

Application

↓

Security Group

↓

Private Subnet

↓

Amazon RDS

↓

Encryption
```

---

# 94. Security Layers in RDS

Amazon RDS security consists of multiple layers.

```
RDS Security

│

├── VPC

├── Private Subnet

├── Security Groups

├── IAM

├── Database Users

├── Encryption

├── SSL

├── Monitoring

└── Logging
```

Never rely on only one security mechanism.

---

# 95. RDS in Private Subnets

Production Architecture

```
Internet

↓

ALB

↓

Application EC2

↓

Private RDS
```

Users never connect directly to the database.

Only the application communicates with the database.

---

# 96. Publicly Accessible Database

Amazon RDS allows:

```
Public Access

YES / NO
```

Production Recommendation

```
NO
```

Development environments may occasionally require public access, but production databases should normally remain private.

---

# 97. Security Groups

Security Groups act as a firewall.

Example

```
Application SG

↓

Allow

↓

Port 3306

↓

RDS SG
```

Instead of allowing IP addresses,

allow Security Group to Security Group communication.

---

# 98. Good Security Group Design

```
Application SG

↓

3306

↓

Database SG
```

Avoid

```
0.0.0.0/0

↓

3306
```

Never expose MySQL directly to the Internet.

---

# 99. Database Users

RDS creates a master database user.

Example

```
admin
```

Production Best Practice

```
Application User

↓

Read/Write

Limited Privileges
```

Do not allow applications to use the master account.

---

# 100. Principle of Least Privilege

Example

Instead of

```
GRANT ALL PRIVILEGES
```

Use

```
SELECT

INSERT

UPDATE
```

Grant only permissions required by the application.

---

# 101. IAM Database Authentication

Normally

```
Username

↓

Password
```

Amazon RDS also supports:

```
IAM Authentication
```

Supported database engines include:

- MySQL
- PostgreSQL

Instead of storing passwords,

applications obtain a temporary authentication token.

---

# 102. IAM Authentication Flow

```
Application

↓

IAM Role

↓

Generate Auth Token

↓

Amazon RDS
```

Benefits

✔ No hardcoded passwords

✔ Temporary credentials

✔ Better security

---

# 103. Database Password vs IAM Authentication

| Password | IAM Authentication |
|-----------|--------------------|
| Static | Temporary |
| Manual Rotation | Automatic Token Generation |
| Can Leak | Reduced Risk |
| Stored in Config Files | Generated When Needed |

---

# 104. Encryption at Rest

Sensitive information should always be encrypted.

Amazon RDS supports:

```
AWS KMS

↓

Encrypted Storage
```

Everything written to storage is encrypted.

---

# 105. What Gets Encrypted?

When encryption is enabled:

✔ Database Storage

✔ Automated Backups

✔ Read Replicas created from encrypted instances

✔ Snapshots

✔ Transaction Logs

---

# 106. AWS KMS Integration

```
Amazon RDS

↓

AWS KMS

↓

Encryption Key

↓

Encrypted Storage
```

AWS manages encryption and decryption using KMS keys.

---

# 107. Can Encryption Be Enabled Later?

Important Interview Question

For many RDS engines, encryption cannot simply be enabled on an existing unencrypted instance.

Typical migration process:

```
Create Snapshot

↓

Copy Snapshot

↓

Enable Encryption

↓

Restore New Database
```

Always plan encryption during database creation whenever possible.

---

# 108. Encryption in Transit

Encryption at rest protects stored data.

Encryption in transit protects network traffic.

```
Application

↓

SSL/TLS

↓

Amazon RDS
```

This prevents attackers from reading traffic while it travels across the network.

---

# 109. SSL/TLS Example

Without SSL

```
Application

↓

Plain Text

↓

Database
```

With SSL

```
Application

↓

Encrypted Traffic

↓

Database
```

Production systems should enforce SSL/TLS wherever supported.

---

# 110. Secrets Manager

Instead of storing passwords in:

```
application.properties
```

Store them in:

```
AWS Secrets Manager
```

Benefits

✔ Automatic Rotation

✔ Encryption

✔ Secure Storage

✔ IAM Integration

---

# 111. Parameter Groups

A Parameter Group stores database configuration values.

Examples

```
max_connections

innodb_buffer_pool_size

log_bin

time_zone
```

Instead of changing configuration on each database,

multiple RDS instances can share the same Parameter Group.

---

# 112. Parameter Group Architecture

```
Parameter Group

↓

Configuration

↓

Multiple Databases
```

Changing a parameter may require either:

- Immediate application (dynamic parameter)
- Database reboot (static parameter)

---

# 113. Option Groups

Some database engines support additional database features.

Example

Oracle

```
Oracle Options

↓

Option Group

↓

Database
```

Option Groups are commonly used for features specific to Oracle or SQL Server.

---

# 114. Enhanced Monitoring

CloudWatch gives database-level metrics.

Enhanced Monitoring also provides operating system metrics.

Examples

- CPU
- Memory
- Processes
- Disk Activity
- Threads

---

# 115. Enhanced Monitoring Architecture

```
Amazon RDS

↓

OS Metrics

↓

CloudWatch Logs

↓

Dashboard
```

Useful for troubleshooting performance problems.

---

# 116. Performance Insights

Performance Insights helps identify:

- Slow SQL
- Database load
- Wait events
- CPU bottlenecks
- Top SQL statements

Instead of guessing,

AWS shows where the database spends its time.

---

# 117. Performance Insights Dashboard

```
Performance Insights

↓

Top Queries

↓

Top Wait Events

↓

DB Load

↓

Recommendations
```

Very useful during production incidents.

---

# 118. CloudWatch Metrics

Important RDS Metrics

```
CPUUtilization

DatabaseConnections

FreeStorageSpace

ReadIOPS

WriteIOPS

ReadLatency

WriteLatency

FreeableMemory

NetworkThroughput
```

These metrics should be monitored continuously.

---

# 119. CloudWatch Alarms

Example

```
CPU

>

80%

↓

Alarm

↓

SNS

↓

Email

↓

DevOps Team
```

Production systems should always have monitoring and alerting.

---

# 120. CloudTrail

CloudTrail records management API activity.

Example

```
CreateDBInstance

DeleteDBInstance

ModifyDBInstance

CreateSnapshot
```

Useful for:

- Auditing
- Compliance
- Security Investigations

---

# 121. Production Security Architecture

```
Users

↓

Application

↓

IAM Role

↓

Secrets Manager

↓

SSL

↓

Private RDS

↓

KMS Encryption

↓

CloudWatch

↓

CloudTrail
```

This is a common enterprise design.

---

# 122. Best Practices

✔ Keep RDS in private subnets.

✔ Use Security Groups instead of public access.

✔ Enable KMS Encryption.

✔ Use SSL/TLS.

✔ Store passwords in Secrets Manager.

✔ Enable Enhanced Monitoring.

✔ Enable Performance Insights.

✔ Create CloudWatch Alarms.

✔ Follow Least Privilege.

---

# 123. Common Mistakes

❌ Public Database

❌ Hardcoded Passwords

❌ No Encryption

❌ No Monitoring

❌ Using Master User

❌ No CloudWatch Alarms

❌ No Audit Logs

---

# 124. Production Scenario

A financial application requires:

- Encryption
- Monitoring
- High Security
- Auditing
- Password Rotation

Architecture

```
Application

↓

IAM Role

↓

Secrets Manager

↓

SSL

↓

Private RDS

↓

KMS

↓

CloudWatch

↓

CloudTrail
```

---

# 125. Interview Questions

## Question 23

Should an RDS database be publicly accessible?

### Answer

No.

Production databases should normally remain in private subnets and only be accessible from trusted application servers.

---

## Question 24

How do Security Groups protect Amazon RDS?

### Answer

Security Groups act as virtual firewalls that control which sources can connect to the database on specific ports.

---

## Question 25

What is IAM Database Authentication?

### Answer

IAM Database Authentication allows supported databases to authenticate users using temporary IAM-generated authentication tokens instead of static passwords.

---

## Question 26

What is encrypted when RDS encryption is enabled?

### Answer

Encryption covers:

- Database storage
- Automated backups
- Snapshots
- Transaction logs
- Read Replicas created from encrypted instances

---

## Question 27

Can you enable encryption on an existing unencrypted RDS instance?

### Answer

Generally, no.

A common approach is to create a snapshot, copy it with encryption enabled, and restore a new encrypted instance.

---

## Question 28

Why use AWS Secrets Manager?

### Answer

Secrets Manager securely stores database credentials, supports automatic rotation, integrates with IAM, and reduces the need to hardcode passwords.

---

## Question 29

What is the purpose of a Parameter Group?

### Answer

A Parameter Group stores database engine configuration settings that can be applied to one or more RDS instances.

---

## Question 30

What is the difference between a Parameter Group and an Option Group?

### Answer

A Parameter Group controls database configuration values.

An Option Group enables additional database engine features, mainly for engines such as Oracle and SQL Server.

---

## Question 31

What is Performance Insights?

### Answer

Performance Insights helps identify database bottlenecks by showing database load, wait events, and the SQL queries consuming the most resources.

---

## Question 32

Which CloudWatch metrics should every DevOps engineer monitor?

### Answer

- CPUUtilization
- DatabaseConnections
- FreeStorageSpace
- FreeableMemory
- ReadLatency
- WriteLatency
- ReadIOPS
- WriteIOPS

---

# 126. Hands-on Labs

## Lab 17

Deploy an RDS instance in private subnets.

---

## Lab 18

Configure Security Groups so only the application server can access the database.

---

## Lab 19

Enable KMS encryption for a new RDS instance.

---

## Lab 20

Enable Performance Insights and analyze SQL performance.

---

## Lab 21

Create CloudWatch alarms for:

- CPU > 80%
- Free Storage < 20 GB
- Database Connections > Threshold

---

## Lab 22

Store RDS credentials in AWS Secrets Manager and connect using the retrieved secret.

---

# 127. One-Page Revision

```
Amazon RDS Security

↓

Private Subnet

↓

Security Groups

↓

IAM Authentication

↓

Secrets Manager

↓

KMS Encryption

↓

SSL/TLS

↓

Parameter Group

↓

Option Group

↓

Enhanced Monitoring

↓

Performance Insights

↓

CloudWatch

↓

CloudTrail
```

Remember:

- Keep RDS private.
- Never expose port 3306 to the Internet.
- Use KMS for encryption.
- Use SSL/TLS for secure connections.
- Store credentials in Secrets Manager.
- Monitor with CloudWatch and Performance Insights.
- Audit with CloudTrail.

---

# Think Like a Senior DevOps Engineer

A secure database is built using **multiple layers of defense**, not a single security feature.

For every production RDS deployment:

1. Deploy the database in **private subnets**.
2. Restrict access using **Security Groups**.
3. Encrypt data at rest with **KMS**.
4. Encrypt network traffic using **SSL/TLS**.
5. Store credentials in **Secrets Manager**.
6. Monitor continuously with **CloudWatch** and **Performance Insights**.
7. Audit all administrative actions with **CloudTrail**.

Enterprise database security is achieved through **defense in depth**, where networking, identity, encryption, monitoring, and auditing work together.

# End of Part 4