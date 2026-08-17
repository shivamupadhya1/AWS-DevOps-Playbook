# Amazon S3

> AWS DevOps Playbook
>
> Volume 1 – Storage
>
> Chapter 22
>
> Amazon S3 – Part 2 (Storage Classes)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand every S3 Storage Class
- Know when to use each storage class
- Compare storage classes
- Understand retrieval charges
- Learn minimum storage duration
- Design cost-optimized storage architectures
- Answer S3 Storage Class interview questions

---

# 27. What are S3 Storage Classes?

Not every file is accessed equally.

Examples:

- Website images → Accessed every second
- Backup files → Accessed once a month
- Legal documents → Accessed once every 5 years

Storing all data in S3 Standard would be expensive.

AWS provides multiple storage classes so you can optimize costs based on access patterns. :contentReference[oaicite:0]{index=0}

---

# 28. S3 Storage Class Family

```
Amazon S3

│

├── S3 Standard

├── S3 Intelligent-Tiering

├── S3 Standard-IA

├── S3 One Zone-IA

├── S3 Glacier Instant Retrieval

├── S3 Glacier Flexible Retrieval

└── S3 Glacier Deep Archive
```

Each storage class targets a different access pattern. :contentReference[oaicite:1]{index=1}

---

# 29. S3 Standard

This is the default storage class.

Best for:

- Frequently accessed data
- Websites
- Mobile apps
- Images
- Videos
- Active datasets
- Application assets

Example:

```
Website

↓

Product Images

↓

S3 Standard
```

Characteristics:

- Millisecond access
- Multi-AZ storage
- High availability
- High durability

---

# 30. When Should You Use S3 Standard?

Choose S3 Standard when:

✔ Files are accessed daily.

✔ Low latency is required.

✔ High throughput is required.

✔ Data is business critical.

Examples:

- Company website
- Application assets
- User uploads
- Logs under active analysis

---

# 31. S3 Intelligent-Tiering

Sometimes you don't know how often data will be accessed.

Example:

```
Employee Documents

↓

Some files opened daily

↓

Some never opened
```

Instead of manually moving objects, Intelligent-Tiering automatically moves eligible objects between access tiers based on access patterns. :contentReference[oaicite:2]{index=2}

---

# 32. How Intelligent-Tiering Works

```
Upload File

↓

Frequent Access Tier

↓

If Not Accessed

↓

Infrequent Access Tier

↓

If Still Not Accessed

↓

Archive Instant Access Tier

↓

(Optional)

Archive Access

↓

(Optional)

Deep Archive Access
```

AWS automatically optimizes storage costs for eligible objects. :contentReference[oaicite:3]{index=3}

---

# 33. When Should You Use Intelligent-Tiering?

Use when:

- Access pattern is unknown
- Access changes over time
- Data lakes
- User uploads
- Analytics
- Long-running projects

Examples:

- HR documents
- Customer uploads
- Research datasets

---

# 34. S3 Standard-IA

IA = **Infrequent Access**

Designed for:

- Files accessed occasionally
- Backups
- Disaster Recovery copies
- Older reports

Characteristics:

- Millisecond retrieval
- Lower storage cost than Standard
- Retrieval charges apply
- Minimum storage duration: **30 days** :contentReference[oaicite:4]{index=4}

---

# 35. Standard-IA Example

```
Application Backup

↓

Created Daily

↓

Rarely Accessed

↓

S3 Standard-IA
```

---

# 36. When Should You Use Standard-IA?

Examples:

- Monthly backups
- Audit reports
- Old project files
- Disaster recovery copies

Avoid it for:

- Frequently accessed files
- Temporary files
- Objects stored for less than 30 days

---

# 37. S3 One Zone-IA

One Zone-IA stores data in a **single Availability Zone** instead of multiple AZs.

Benefits:

- Lower storage cost

Trade-off:

- Not resilient to the loss of that Availability Zone. :contentReference[oaicite:5]{index=5}

---

# 38. When Should You Use One Zone-IA?

Suitable for:

- Re-creatable data
- Temporary backups
- Secondary copies
- Build artifacts
- Cache files

Not suitable for:

- Production databases
- Financial records
- Critical customer data

---

# 39. S3 Glacier Instant Retrieval

Designed for:

- Long-term storage
- Rarely accessed data
- Millisecond retrieval

Examples:

- Medical records
- Archived contracts
- Compliance documents

Characteristics:

- Archive pricing
- Instant retrieval
- Minimum storage duration: **90 days** :contentReference[oaicite:6]{index=6}

---

# 40. Glacier Instant Retrieval Example

```
Hospital Records

↓

Archive

↓

Occasionally Needed

↓

Immediate Access Required

↓

Glacier Instant Retrieval
```

---

# 41. S3 Glacier Flexible Retrieval

Formerly called **Amazon Glacier**.

Designed for:

- Long-term archives
- Backup storage
- Rare access

Retrieval time depends on the retrieval option selected and can range from minutes to hours. :contentReference[oaicite:7]{index=7}

Examples:

- Yearly backups
- Compliance archives
- Historical logs

---

# 42. Glacier Flexible Retrieval Example

```
Company Backup

↓

Archive

↓

Restore Only During Disaster

↓

Glacier Flexible Retrieval
```

---

# 43. S3 Glacier Deep Archive

The lowest-cost S3 storage class.

Designed for:

- Data that is almost never accessed
- Long-term retention
- Digital preservation

Retrieval typically takes hours.

Minimum storage duration:

**180 days**. :contentReference[oaicite:8]{index=8}

---

# 44. Glacier Deep Archive Example

```
Legal Documents

↓

10 Years Retention

↓

Rarely Accessed

↓

Deep Archive
```

---

# 45. Storage Class Comparison

| Storage Class | Access Pattern | Retrieval Speed | Multi-AZ | Typical Use |
|---------------|---------------|----------------|----------|-------------|
| Standard | Frequent | Milliseconds | Yes | Websites, applications |
| Intelligent-Tiering | Unknown | Milliseconds | Yes | Mixed workloads |
| Standard-IA | Monthly | Milliseconds | Yes | Backups |
| One Zone-IA | Monthly | Milliseconds | No | Re-creatable data |
| Glacier Instant Retrieval | Rare | Milliseconds | Yes | Archive with instant access |
| Glacier Flexible Retrieval | Very Rare | Minutes to Hours | Yes | Backup archives |
| Glacier Deep Archive | Almost Never | Hours | Yes | Long-term archive |

Data summarized from AWS storage class documentation. :contentReference[oaicite:9]{index=9}

---

# 46. Production Decision Examples

## Website Images

```
Website

↓

Millions of Requests

↓

S3 Standard
```

---

## Unknown Access Pattern

```
Research Files

↓

Intelligent-Tiering
```

---

## Monthly Backup

```
Daily Backup

↓

Retain 1 Year

↓

Standard-IA
```

---

## Temporary Build Files

```
Jenkins Build

↓

Artifacts

↓

One Zone-IA
```

---

## Medical Records

```
Hospital

↓

Archive

↓

Glacier Instant Retrieval
```

---

## Disaster Recovery Backup

```
Database Backup

↓

Glacier Flexible Retrieval
```

---

## 10-Year Compliance Archive

```
Audit Records

↓

Deep Archive
```

---

# 47. Cost Optimization Strategy

Example lifecycle:

```
Upload

↓

S3 Standard

↓

30 Days

↓

Standard-IA

↓

90 Days

↓

Glacier Instant Retrieval

↓

1 Year

↓

Deep Archive
```

This minimizes storage cost while maintaining appropriate accessibility. :contentReference[oaicite:10]{index=10}

---

# 48. Best Practices

✔ Store active data in S3 Standard.

✔ Use Intelligent-Tiering when access patterns are unpredictable.

✔ Archive old data automatically with Lifecycle Rules.

✔ Avoid One Zone-IA for irreplaceable data.

✔ Understand retrieval charges before selecting IA or Glacier classes.

✔ Review storage class usage periodically.

---

# 49. Common Mistakes

❌ Storing active application data in Glacier.

❌ Using One Zone-IA for critical production files.

❌ Forgetting minimum storage duration charges.

❌ Ignoring retrieval costs.

❌ Keeping all data in Standard forever.

---

# 50. Production Scenario

A bank stores:

```
Current Statements

↓

S3 Standard

------------------

1-Year Statements

↓

Standard-IA

------------------

5-Year Statements

↓

Glacier Instant Retrieval

------------------

10-Year Compliance Records

↓

Deep Archive
```

This balances performance, compliance, and cost.

---

# 51. Interview Questions

## Question 9

What is the default S3 storage class?

### Answer

S3 Standard.

---

## Question 10

Which storage class automatically moves objects based on access patterns?

### Answer

S3 Intelligent-Tiering.

---

## Question 11

What is the difference between Standard-IA and One Zone-IA?

### Answer

Standard-IA stores data across multiple Availability Zones, while One Zone-IA stores data in a single Availability Zone and is intended for re-creatable data.

---

## Question 12

Which storage class should be used for frequently accessed application data?

### Answer

S3 Standard.

---

## Question 13

Which storage class is the least expensive?

### Answer

S3 Glacier Deep Archive.

---

## Question 14

Which storage class should be used for backups that are rarely restored but may need retrieval within minutes or hours?

### Answer

S3 Glacier Flexible Retrieval.

---

## Question 15

Can Glacier Flexible Retrieval objects be accessed immediately?

### Answer

No.

Archived objects must first be restored before they can be accessed.

---

## Question 16

What is the minimum storage duration for:

- Standard-IA
- Glacier Instant Retrieval
- Glacier Deep Archive

### Answer

- Standard-IA → 30 days
- Glacier Instant Retrieval → 90 days
- Glacier Deep Archive → 180 days :contentReference[oaicite:11]{index=11}

---

# 52. Hands-on Labs

## Lab 6

Create objects in:

- Standard
- Standard-IA
- Intelligent-Tiering

Compare the storage classes.

---

## Lab 7

Design lifecycle transitions from Standard to Glacier.

---

## Lab 8

Choose the correct storage class for:

- Website images
- Monthly backups
- Audit logs
- Medical records
- Legal archives

---

## Lab 9

Design a storage strategy for an e-commerce application.

---

## Lab 10

Compare all storage classes in a table and explain the trade-offs.

---

# 53. One-Page Revision

```
Frequently Used

↓

S3 Standard

----------------

Unknown Access

↓

Intelligent-Tiering

----------------

Monthly Access

↓

Standard-IA

----------------

Re-creatable

↓

One Zone-IA

----------------

Archive + Instant

↓

Glacier Instant Retrieval

----------------

Archive

↓

Glacier Flexible Retrieval

----------------

Long-Term Archive

↓

Deep Archive
```

Remember:

- Standard → Active workloads
- Intelligent-Tiering → Unknown access
- Standard-IA → Infrequent access
- One Zone-IA → Re-creatable data
- Glacier Instant Retrieval → Rare but instant access
- Glacier Flexible Retrieval → Archive with restore
- Deep Archive → Lowest-cost long-term storage

---

# Think Like a Production Engineer

Storage design is about matching **business requirements** to the correct storage class.

Before choosing a storage class, ask:

1. How often is the data accessed?
2. How quickly must it be retrieved?
3. Can the data be recreated?
4. What are the retention requirements?
5. What is the acceptable storage cost?

The right storage class can significantly reduce costs without compromising application requirements.

# End of Part 2
