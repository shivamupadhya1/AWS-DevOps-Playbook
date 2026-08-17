# Amazon RDS

> AWS DevOps Playbook
>
> Volume 1 – Databases
>
> Chapter 23
>
> Amazon RDS – Part 3 (Read Replicas, Scaling & Production Architectures)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Read Replicas
- Learn why Read Replicas are needed
- Understand Read Scaling
- Learn Asynchronous Replication
- Understand Replica Lag
- Learn Cross-Region Read Replicas
- Understand Promotion
- Compare Read Replica vs Multi-AZ
- Learn Production Architectures
- Answer Interview Questions

---

# 63. Why Read Replicas?

Suppose your application receives:

```
1 Million Users

↓

Database

↓

Very High Read Traffic
```

The database becomes slow.

Example:

```
SELECT

SELECT

SELECT

SELECT

SELECT

...

Thousands per second
```

CPU utilization reaches:

```
95%
```

Application response time increases.

---

# 64. Solution

Instead of sending every request to one database:

```
Application

↓

Primary Database
```

We create additional databases.

These are called:

```
Read Replicas
```

---

# 65. What is a Read Replica?

A Read Replica is a **read-only copy** of the primary database.

```
Primary Database

↓

Asynchronous Replication

↓

Read Replica
```

Applications send:

- READ queries → Replica
- WRITE queries → Primary

---

# 66. Read Replica Architecture

```
                Application

                     │

     ┌───────────────┴───────────────┐

     │                               │

READ Queries                    WRITE Queries

     │                               │

Read Replica                  Primary Database
```

This greatly reduces load on the primary database.

---

# 67. Real Production Example

E-commerce website:

```
Users

↓

Browse Products

↓

SELECT Queries

↓

Read Replica

----------------------

Checkout

↓

INSERT Order

↓

Primary Database
```

Most traffic is reading product data.

---

# 68. Read Scaling

Suppose:

```
Primary

↓

10,000 Reads/sec
```

CPU becomes:

```
95%
```

Solution:

```
Primary

↓

Replica 1

Replica 2

Replica 3
```

Now read traffic is distributed.

---

# 69. Replication Process

Every write goes to:

```
Primary

↓

Binary Logs

↓

Replica

↓

Replay Changes
```

Replication is automatic.

---

# 70. Asynchronous Replication

Unlike Multi-AZ,

Read Replicas use:

```
Asynchronous Replication
```

Meaning:

```
Write

↓

Primary

↓

Success Returned

↓

Replica Updated Later
```

The primary does **not** wait for the replica before acknowledging the write.

---

# 71. Why Asynchronous?

Because:

```
Application

↓

Fast Writes
```

If the primary waited for replicas, write performance would decrease.

---

# 72. Replica Lag

Since replication is asynchronous:

```
Primary Updated

↓

Replica

↓

Few Seconds Later
```

This delay is called:

```
Replica Lag
```

---

# 73. Replica Lag Example

Customer updates profile.

```
UPDATE Name

↓

Primary Updated
```

Immediately afterwards:

```
Read Replica

↓

Old Value
```

After replication catches up:

```
New Value
```

Applications should account for this eventual consistency.

---

# 74. Monitoring Replica Lag

CloudWatch provides metrics to monitor replication health.

Important metric:

```
ReplicaLag
```

High values indicate that replicas are falling behind.

Possible reasons:

- Heavy writes
- Network latency
- Large transactions
- Resource bottlenecks

---

# 75. Read Replica Promotion

Suppose:

```
Primary Database

↓

Destroyed
```

You can promote a Read Replica.

```
Replica

↓

Promote

↓

Independent Database
```

After promotion:

- Replication stops.
- The promoted instance becomes a standalone database.

---

# 76. Cross-Region Read Replica

Example:

```
Mumbai

↓

Primary

↓

Replication

↓

Singapore

↓

Replica
```

Benefits:

- Disaster Recovery
- Local reads
- Reporting
- Global applications

---

# 77. Global Read Architecture

```
USA Users

↓

USA Replica

----------------

Europe Users

↓

Europe Replica

----------------

India Users

↓

Primary
```

Users read from the closest replica.

---

# 78. Reporting Database

Production applications often generate reports.

Instead of running reports on the primary:

```
Primary

↓

Read Replica

↓

BI Reports
```

Benefits:

- Faster reports
- Less impact on production
- Better user experience

---

# 79. Analytics Example

```
Application

↓

Primary

↓

Replica

↓

Power BI

↓

Dashboard
```

Reporting queries no longer affect transactional workloads.

---

# 80. Read Replica vs Multi-AZ

This is one of the most common interview questions.

| Feature | Read Replica | Multi-AZ |
|----------|--------------|-----------|
| Purpose | Read Scaling | High Availability |
| Replication | Asynchronous | Synchronous |
| Read Traffic | Yes | No |
| Failover | Manual promotion | Automatic |
| Number of Replicas | Multiple | One standby per deployment |
| Performance | Improves reads | Does not improve read performance |

---

# 81. When to Use Multi-AZ

Use when you need:

- High Availability
- Automatic Failover
- Production Reliability
- Business Continuity

---

# 82. When to Use Read Replicas

Use when you need:

- Read Scaling
- Analytics
- Reporting
- Global Read Performance
- Reduced Load on Primary

---

# 83. Can You Use Both?

Yes.

A common production architecture combines both.

```
                Application

                     │

          Write Queries

                     │

              Primary (Multi-AZ)

                     │

        Synchronous Standby

                     │

      Asynchronous Read Replicas
```

Benefits:

- High Availability
- Read Scaling
- Disaster Recovery

---

# 84. Production Banking Architecture

```
Internet

↓

ALB

↓

Application

↓

Primary RDS

↓

Standby (Multi-AZ)

↓

Read Replica

↓

Reporting
```

Customer transactions always go to the primary.

Reports use the replica.

---

# 85. E-Commerce Architecture

```
Users

↓

Application

↓

Primary

↓

Replica 1

↓

Replica 2

↓

Replica 3
```

Reads are distributed.

Writes go only to the primary.

---

# 86. Gaming Architecture

```
Players

↓

Primary Database

↓

Regional Read Replicas

↓

Leaderboards

↓

Statistics
```

Read Replicas reduce latency for players around the world.

---

# 87. Best Practices

✔ Use Multi-AZ for High Availability.

✔ Use Read Replicas for Read Scaling.

✔ Monitor Replica Lag.

✔ Do not send writes to Read Replicas.

✔ Use Cross-Region Replicas for Disaster Recovery and global reads.

✔ Test replica promotion regularly.

---

# 88. Common Mistakes

❌ Assuming Read Replicas provide automatic failover.

❌ Sending write queries to replicas.

❌ Ignoring Replica Lag.

❌ Using replicas as backups.

❌ Expecting Multi-AZ to improve read performance.

---

# 89. Production Scenario

An online shopping platform receives:

```
500,000 Users
```

Traffic:

```
95%

↓

SELECT

5%

↓

INSERT/UPDATE
```

Architecture:

```
Application

↓

Primary

↓

Read Replica 1

↓

Read Replica 2

↓

Read Replica 3
```

Read requests are load-balanced across replicas.

---

# 90. Interview Questions

## Question 15

What is a Read Replica?

### Answer

A Read Replica is a read-only copy of a primary RDS database used to offload read traffic and improve application scalability.

---

## Question 16

Does a Read Replica support write operations?

### Answer

No.

Applications should send write operations only to the primary database.

---

## Question 17

What type of replication does a Read Replica use?

### Answer

Asynchronous replication.

---

## Question 18

What is Replica Lag?

### Answer

Replica Lag is the delay between a successful write on the primary database and the time that change becomes visible on the Read Replica.

---

## Question 19

Can a Read Replica be promoted?

### Answer

Yes.

A Read Replica can be promoted to become an independent standalone database.

---

## Question 20

Can Read Replicas improve database write performance?

### Answer

No.

They improve read scalability only.

---

## Question 21

What is the difference between Multi-AZ and Read Replicas?

### Answer

- Multi-AZ provides High Availability using synchronous replication and automatic failover.
- Read Replicas provide Read Scaling using asynchronous replication.

---

## Question 22

Can you have both Multi-AZ and Read Replicas?

### Answer

Yes.

Many production systems use Multi-AZ for availability and Read Replicas for scaling.

---

# 91. Hands-on Labs

## Lab 12

Create a Read Replica for a MySQL RDS instance.

---

## Lab 13

Connect to the Read Replica and execute SELECT queries.

---

## Lab 14

Generate heavy read traffic and monitor Replica Lag.

---

## Lab 15

Promote a Read Replica to a standalone database.

---

## Lab 16

Create a Cross-Region Read Replica (where supported by the chosen engine).

---

# 92. One-Page Revision

```
Application

↓

Primary Database

↓

Asynchronous Replication

↓

Read Replica

↓

Read Queries

-------------------------

Multi-AZ

↓

Synchronous Replication

↓

Standby

↓

Automatic Failover
```

Remember:

- Read Replica = Read Scaling.
- Multi-AZ = High Availability.
- Read Replica = Asynchronous replication.
- Multi-AZ = Synchronous replication.
- Replica Lag is normal.
- Replicas are read-only.
- Replicas can be promoted manually.

---

# Think Like a Production Engineer

A common mistake is trying to solve every database problem with a larger instance.

Instead:

- Use **vertical scaling** (larger DB instance) when CPU or memory is consistently insufficient.
- Use **Read Replicas** when read traffic is the bottleneck.
- Use **Multi-AZ** to protect against infrastructure failures.
- Monitor **Replica Lag** and route read traffic appropriately.
- Separate transactional workloads from reporting and analytics.

A well-designed production database architecture is **available, scalable, resilient, and optimized for both reads and writes**.

# End of Part 3