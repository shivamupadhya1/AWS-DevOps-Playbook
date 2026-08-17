# Amazon RDS

> AWS DevOps Playbook
>
> Volume 1 – Databases
>
> Chapter 23
>
> Amazon RDS – Part 5 (Performance Optimization, Scaling & Cost Optimization)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand RDS Performance Optimization
- Learn Vertical Scaling
- Learn Horizontal Scaling
- Understand Connection Pooling
- Learn Performance Insights
- Understand Slow Query Logs
- Learn Parameter Optimization
- Understand Storage Optimization
- Learn Cost Optimization
- Design Production Performance Architectures
- Answer Interview Questions

---

# 128. Why Database Performance Matters

A slow database affects the entire application.

Example:

```
User

↓

Application

↓

Database (Slow)

↓

Application Slow

↓

Poor User Experience
```

Most enterprise performance issues originate from:

- Slow SQL queries
- Insufficient CPU
- Memory shortage
- Poor indexing
- Too many database connections
- Storage bottlenecks

---

# 129. Performance Bottlenecks

```
Database

│

├── CPU

├── Memory

├── Storage IOPS

├── Network

├── Locks

├── Slow Queries

└── Too Many Connections
```

As a DevOps Engineer, identify the bottleneck before scaling.

---

# 130. Vertical Scaling

Vertical scaling means increasing the size of the DB instance.

Example

```
db.t3.medium

↓

db.m6i.large

↓

db.r6g.xlarge
```

Benefits:

✔ More CPU

✔ More RAM

✔ Better Performance

---

# 131. When to Use Vertical Scaling

Suitable when:

- CPU is consistently high.
- Memory usage is high.
- The database is under-provisioned.
- Workload has increased.

Monitor before scaling.

---

# 132. Horizontal Scaling

Horizontal scaling in RDS is achieved using **Read Replicas**.

```
Primary

↓

Replica 1

Replica 2

Replica 3
```

Read queries are distributed.

Write queries still go to the primary.

---

# 133. Which Scaling Should You Choose?

| Scenario | Solution |
|----------|----------|
| High CPU | Vertical Scaling |
| Memory Shortage | Vertical Scaling |
| Too Many Read Queries | Read Replicas |
| High Availability | Multi-AZ |
| Storage Full | Storage Auto Scaling |

---

# 134. Connection Pooling

Every database connection consumes:

- Memory
- CPU
- Network Resources

Bad Practice

```
1000 Users

↓

1000 Database Connections
```

Good Practice

```
1000 Users

↓

Connection Pool

↓

50 Database Connections
```

---

# 135. Benefits of Connection Pooling

✔ Faster Response

✔ Reduced Memory Usage

✔ Lower CPU Usage

✔ Better Throughput

Popular Java connection pools:

- HikariCP
- Apache DBCP
- c3p0

Spring Boot uses **HikariCP** by default.

---

# 136. Slow Query Logs

Sometimes the database is healthy,

but a query takes:

```
15 Seconds
```

instead of

```
50 ms
```

Enable slow query logging.

Example:

```
SELECT *

FROM orders

WHERE customer_name='Rahul'
```

Without proper indexing, the query may scan the entire table.

---

# 137. Why Slow Queries Occur

Common reasons:

- Missing indexes
- Full table scans
- Large joins
- Poor query design
- Too much data

Scaling the database won't fix poorly written SQL.

---

# 138. Indexing

Example

Without Index

```
1 Million Rows

↓

Sequential Scan
```

With Index

```
Index

↓

Locate Row

↓

Milliseconds
```

Indexes dramatically improve read performance.

---

# 139. Parameter Optimization

Examples of tunable parameters:

```
max_connections

work_mem

shared_buffers

sort_buffer_size
```

Use Parameter Groups to optimize configuration.

Always test changes before production rollout.

---

# 140. Performance Insights

Performance Insights displays:

```
Database Load

↓

Top SQL

↓

Wait Events

↓

Top Users

↓

CPU Usage
```

Useful for identifying bottlenecks quickly.

---

# 141. CloudWatch Performance Metrics

Monitor:

```
CPUUtilization

↓

FreeableMemory

↓

DatabaseConnections

↓

ReadIOPS

↓

WriteIOPS

↓

ReadLatency

↓

WriteLatency

↓

DiskQueueDepth
```

Create alarms for abnormal values.

---

# 142. Storage Performance

Storage performance depends on:

- Storage Type
- Provisioned IOPS
- Throughput
- Latency

Example

```
Provisioned IOPS

↓

Fast Storage

↓

Low Latency
```

---

# 143. Storage Auto Scaling

Storage should never become full.

Example

```
90%

↓

95%

↓

Automatically Increase Storage
```

This prevents outages caused by disk exhaustion.

---

# 144. Optimizing Read Workloads

Architecture

```
Users

↓

Application

↓

Read Replica

↓

Read Queries
```

This reduces CPU utilization on the primary database.

---

# 145. Optimizing Write Workloads

Write operations always go to:

```
Primary Database
```

Improve write performance by:

- Faster instance class
- Better storage
- Query optimization
- Proper indexing

Read Replicas do **not** improve write performance.

---

# 146. Database Caching

Frequently accessed data should not always hit the database.

Architecture

```
Application

↓

Redis

↓

Cache Hit

↓

Return Data

----------------------

Cache Miss

↓

Amazon RDS
```

Benefits:

- Lower database load
- Faster response time
- Reduced latency

Amazon ElastiCache (Redis/Memcached) is commonly used.

---

# 147. Query Optimization Process

```
Slow Query

↓

Execution Plan

↓

Identify Bottleneck

↓

Create Index

↓

Optimize Query

↓

Retest
```

Never optimize without measuring.

---

# 148. Cost Optimization

Ways to reduce cost:

✔ Choose the correct instance size.

✔ Use gp3 where appropriate.

✔ Stop development databases when not in use.

✔ Delete unused snapshots.

✔ Use Reserved Instances for long-running production workloads.

✔ Enable Storage Auto Scaling instead of overprovisioning.

---

# 149. Performance Optimization Checklist

```
✔ CPU < 70%

✔ Memory Available

✔ Storage Healthy

✔ Read Latency Low

✔ Write Latency Low

✔ Queries Indexed

✔ Replica Lag Normal

✔ CloudWatch Alarms Configured

✔ Performance Insights Enabled
```

---

# 150. Production Performance Architecture

```
Users

↓

Application

↓

Connection Pool

↓

Primary RDS

↓

Read Replicas

↓

Redis Cache

↓

CloudWatch

↓

Performance Insights
```

This architecture supports:

- High throughput
- Fast response
- Monitoring
- Scalability

---

# 151. Common Performance Problems

❌ Missing indexes

❌ Thousands of database connections

❌ Small DB instance

❌ Slow storage

❌ No monitoring

❌ Heavy reporting on primary database

❌ Large transactions

❌ Long-running queries

---

# 152. Best Practices

✔ Enable Performance Insights.

✔ Monitor CloudWatch metrics.

✔ Use HikariCP connection pooling.

✔ Create indexes carefully.

✔ Use Read Replicas for read-heavy workloads.

✔ Use Multi-AZ for availability.

✔ Review slow query logs regularly.

✔ Test performance after changes.

---

# 153. Production Scenario

An online banking application reports:

```
CPU

95%
```

Investigation reveals:

```
Heavy SELECT Queries
```

Solution:

```
Primary

↓

Read Replica

↓

Reporting

↓

Performance Restored
```

If CPU remains high after offloading reads:

- Scale vertically.
- Optimize SQL.
- Review indexes.

---

# 154. Interview Questions

## Question 33

How do you improve Amazon RDS performance?

### Answer

- Optimize SQL queries.
- Create indexes.
- Enable Performance Insights.
- Monitor CloudWatch.
- Scale vertically.
- Use Read Replicas.
- Enable connection pooling.

---

## Question 34

Does increasing CPU always solve database problems?

### Answer

No.

Many issues are caused by poor SQL, missing indexes, or excessive connections rather than insufficient compute.

---

## Question 35

How do Read Replicas improve performance?

### Answer

They offload read traffic from the primary database, allowing it to focus on write operations.

---

## Question 36

What is connection pooling?

### Answer

Connection pooling reuses a limited number of database connections instead of opening a new connection for every request, reducing resource consumption and improving performance.

---

## Question 37

Which metrics should you monitor for performance?

### Answer

- CPUUtilization
- FreeableMemory
- DatabaseConnections
- ReadLatency
- WriteLatency
- ReadIOPS
- WriteIOPS
- DiskQueueDepth

---

## Question 38

What is the purpose of Performance Insights?

### Answer

Performance Insights identifies database bottlenecks by displaying database load, top SQL statements, and wait events.

---

## Question 39

Why are indexes important?

### Answer

Indexes reduce the amount of data scanned during queries, improving read performance significantly.

---

## Question 40

Should you use Redis with Amazon RDS?

### Answer

Yes, for frequently accessed data. Redis reduces database load and improves application response time.

---

# 155. Hands-on Labs

## Lab 23

Enable Performance Insights and identify the top SQL query.

---

## Lab 24

Create an index on a large table and compare query execution times.

---

## Lab 25

Generate read traffic and offload it to a Read Replica.

---

## Lab 26

Configure HikariCP connection pooling in a Spring Boot application.

---

## Lab 27

Create CloudWatch alarms for:

- CPU > 80%
- Read Latency > Threshold
- Free Storage < Threshold

---

## Lab 28

Benchmark query performance before and after optimization.

---

# 156. One-Page Revision

```
Performance

↓

CPU

Memory

Connections

Storage

↓

Performance Insights

↓

CloudWatch

↓

Indexes

↓

Connection Pool

↓

Read Replica

↓

Redis Cache

↓

Vertical Scaling

↓

Storage Optimization
```

Remember:

- Vertical Scaling = More CPU & RAM.
- Read Replicas = Read Scaling.
- Performance Insights = SQL analysis.
- CloudWatch = Infrastructure metrics.
- Connection Pool = Better resource usage.
- Redis = Reduce database load.
- Indexes = Faster queries.

---

# Think Like a Senior DevOps Engineer

When a production database becomes slow:

1. **Do not scale immediately.**
2. Identify whether the bottleneck is CPU, memory, storage, network, or SQL.
3. Use **Performance Insights** to locate expensive queries.
4. Check execution plans and create indexes where appropriate.
5. Offload read traffic using **Read Replicas**.
6. Add caching (Redis) for frequently accessed data.
7. Scale the database only after optimization.

The best database architecture is not the one with the **largest instance**, but the one that delivers **maximum performance with efficient resource utilization and optimized cost**.

# End of Part 5