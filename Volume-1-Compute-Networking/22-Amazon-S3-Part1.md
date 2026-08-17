# Amazon S3

> AWS DevOps Playbook
>
> Volume 1 – Storage
>
> Chapter 22
>
> Amazon S3 – Part 1 (Fundamentals)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Amazon S3
- Explain Object Storage
- Understand Buckets and Objects
- Learn S3 Architecture
- Understand Durability vs Availability
- Learn Bucket Naming Rules
- Understand S3 Data Model
- Know S3 Limits
- Identify Common Use Cases
- Answer S3 interview questions

---

# 1. Introduction to Amazon S3

Amazon S3 stands for **Amazon Simple Storage Service**.

It is a fully managed **Object Storage** service provided by AWS.

Unlike EBS or EFS, S3 stores **objects** instead of blocks or files.

Amazon S3 is designed for:

- Massive scalability
- High durability
- High availability
- Low maintenance
- Secure data storage

It is one of the oldest and most widely used AWS services.

---

# 2. What is Object Storage?

There are three primary storage types in AWS:

| Storage Type | AWS Service | Stores |
|--------------|-------------|--------|
| Block Storage | Amazon EBS | Blocks |
| File Storage | Amazon EFS | Files |
| Object Storage | Amazon S3 | Objects |

An object contains:

- Data
- Metadata
- Unique Key

Example:

```
holiday-photo.jpg

↓

Object

↓

Data

+

Metadata

+

Key
```

Unlike file systems, S3 does not use directories in the traditional sense.

---

# 3. Real World Example

Suppose a company runs an e-commerce website.

The application stores:

- Product images
- Videos
- PDF invoices
- User uploads
- Website assets

Instead of storing these files on EC2:

```
Customer Upload

↓

Application

↓

Amazon S3 Bucket

↓

Store Object
```

Benefits:

- Unlimited scalability
- Lower storage management overhead
- High durability

---

# 4. What is a Bucket?

A **Bucket** is the top-level container in Amazon S3.

Think of it as a logical container for storing objects.

Example:

```
Bucket

company-images

↓

Object

logo.png

↓

Object

banner.jpg

↓

Object

invoice.pdf
```

Every object must belong to exactly one bucket.

---

# 5. What is an Object?

An object is the actual file stored in S3.

Examples:

```
resume.pdf

video.mp4

backup.zip

image.png

logs.tar.gz
```

Each object consists of:

```
Object

↓

Data

↓

Metadata

↓

Key

↓

Version ID (if versioning enabled)
```

---

# 6. What is a Key?

Every object has a unique identifier called the **Object Key**.

Example:

```
Bucket

company-data

↓

reports/2026/january.pdf
```

Here:

Bucket:

```
company-data
```

Key:

```
reports/2026/january.pdf
```

The key uniquely identifies the object inside the bucket.

---

# 7. Understanding Prefixes

Many people think S3 has folders.

Technically, it does not.

Example:

```
reports/

reports/2026/

reports/2026/january.pdf
```

The slash (`/`) is simply part of the object key.

AWS Console displays these prefixes like folders for convenience.

---

# 8. S3 Architecture

```
Application

↓

Amazon S3

↓

Bucket

↓

Objects

↓

Stored Across Multiple Devices

↓

Multiple Availability Zones
```

AWS automatically manages:

- Storage hardware
- Replication across Availability Zones
- Disk failures
- Scaling
- Capacity

You never manage disks or file systems.

---

# 9. How S3 Stores Data

When an object is uploaded:

```
Upload File

↓

Amazon S3

↓

Object Created

↓

Replicated Across Multiple Devices

↓

Stored Durably
```

AWS handles redundancy behind the scenes.

---

# 10. Durability vs Availability

These two concepts are often confused.

### Durability

**Durability** means your data is unlikely to be lost.

Amazon S3 Standard is designed for **99.999999999% (11 nines) durability**.

Example:

If you store 10 million objects, statistically you might expect to lose only a tiny fraction over an extremely long period due to the durability design.

---

### Availability

Availability means how often you can access your data.

S3 Standard is designed for **99.99% availability**.

Example:

Even if a temporary service disruption occurs, your data is still stored durably.

---

# 11. Durability vs Availability Example

Imagine a library.

Durability:

The books are safely stored and protected from damage.

Availability:

The library is open so you can borrow the books.

A library may be temporarily closed (availability issue), but the books still exist (durability).

---

# 12. S3 Storage Classes Overview

Amazon S3 provides multiple storage classes for different access patterns.

Main storage classes include:

- S3 Standard
- S3 Intelligent-Tiering
- S3 Standard-IA
- S3 One Zone-IA
- S3 Glacier Instant Retrieval
- S3 Glacier Flexible Retrieval
- S3 Glacier Deep Archive

We'll study these in Part 2.

---

# 13. Bucket Naming Rules

Bucket names must:

✔ Be globally unique.

✔ Be between 3 and 63 characters.

✔ Use lowercase letters.

✔ Use numbers.

✔ Use hyphens (`-`).

Example:

```
company-backups

my-photo-storage

devops-logs-2026
```

Invalid examples:

```
MyBucket

backup_bucket

Company@Data
```

---

# 14. Global Namespace

Bucket names are unique across all AWS accounts.

Example:

If someone already owns:

```
mycompany-data
```

You cannot create another bucket with the same name.

This is because bucket names exist in a global namespace.

---

# 15. S3 Object Size Limits

Maximum object size:

**5 TB**

Maximum single PUT upload:

**5 GB**

For objects larger than 5 GB, use **Multipart Upload**.

We'll cover Multipart Upload in a later chapter.

---

# 16. Metadata

Every object has metadata.

Example:

```
File Name

image.jpg

Content-Type

image/jpeg

Size

2 MB

Last Modified

17 Aug 2026
```

Metadata helps applications understand how to process the object.

---

# 17. User Metadata

You can also add your own metadata.

Example:

```
Department = Finance

Project = Payroll

Owner = DevOps Team
```

Applications can use custom metadata for automation and organization.

---

# 18. Common S3 Use Cases

Amazon S3 is commonly used for:

- Image Storage
- Video Storage
- Log Storage
- Application Backups
- Static Website Hosting
- Data Lakes
- Big Data Analytics
- CI/CD Artifacts
- Machine Learning Datasets
- Software Downloads

---

# 19. S3 in a Typical Architecture

```
User

↓

Application Load Balancer

↓

EC2 / ECS

↓

Amazon S3

↓

Store Images

↓

Store Documents

↓

Store Videos
```

---

# 20. Benefits of Amazon S3

✔ Virtually unlimited storage.

✔ Highly durable.

✔ Highly available.

✔ Secure.

✔ Pay only for what you use.

✔ Integrates with almost every AWS service.

✔ Supports lifecycle management.

✔ Supports versioning.

✔ Supports encryption.

---

# 21. Best Practices

✔ Use meaningful bucket names.

✔ Enable versioning for important buckets.

✔ Encrypt sensitive data.

✔ Block public access unless intentionally required.

✔ Organize objects using prefixes.

✔ Use lifecycle policies for cost optimization.

---

# 22. Common Mistakes

❌ Making buckets public unintentionally.

❌ Storing application secrets in S3.

❌ Ignoring versioning.

❌ Using random bucket names without standards.

❌ Uploading large files without Multipart Upload.

---

# 23. Production Scenario

A company stores customer invoices.

Architecture:

```
Customer

↓

Application

↓

Amazon S3

↓

Invoice PDF

↓

Lifecycle Policy

↓

Archive After 90 Days
```

Benefits:

- Lower storage costs
- Secure storage
- Easy retrieval
- Long-term retention

---

# 24. Interview Questions

## Question 1

What is Amazon S3?

### Answer

Amazon S3 is a fully managed object storage service that stores data as objects inside buckets. It is designed for high durability, scalability, and availability.

---

## Question 2

What is the difference between EBS, EFS, and S3?

### Answer

- EBS provides block storage for EC2 instances.
- EFS provides shared file storage using NFS.
- S3 provides object storage accessible through APIs.

---

## Question 3

What is a bucket?

### Answer

A bucket is a logical container used to store objects in Amazon S3.

---

## Question 4

What is an object key?

### Answer

The object key is the unique identifier of an object within a bucket. It includes the full path-like prefix, such as `reports/2026/january.pdf`.

---

## Question 5

Does Amazon S3 have folders?

### Answer

No.

S3 does not have real directories. The folder structure shown in the AWS Console is created using prefixes in object keys.

---

## Question 6

What is the maximum object size in Amazon S3?

### Answer

An individual object can be up to **5 TB**. For uploads larger than **5 GB**, Multipart Upload should be used.

---

## Question 7

What is the durability of Amazon S3 Standard?

### Answer

Amazon S3 Standard is designed for **99.999999999% (11 nines) durability**.

---

## Question 8

Why are bucket names globally unique?

### Answer

Bucket names are part of a global namespace so each bucket name must be unique across all AWS accounts.

---

# 25. Hands-on Labs

## Lab 1

Create an S3 bucket using the AWS Management Console.

---

## Lab 2

Upload:

- Image
- PDF
- ZIP file

Observe their metadata.

---

## Lab 3

Create objects using prefixes:

```
images/

documents/

backups/
```

Observe how the AWS Console displays them as folders.

---

## Lab 4

Upload a file and inspect:

- Object Key
- Metadata
- Size
- Last Modified

---

## Lab 5

Design an S3 architecture for storing application images, logs, and backups.

---

# 26. One-Page Revision

```
Amazon S3

↓

Bucket

↓

Object

↓

Key

↓

Metadata

↓

Storage Class

↓

Multiple Availability Zones
```

Remember:

- S3 is Object Storage.
- Bucket = Container.
- Object = File.
- Key = Unique Identifier.
- Prefixes simulate folders.
- Maximum Object Size = 5 TB.
- Multipart Upload for files > 5 GB.
- 11 Nines Durability.
- Globally unique bucket names.

---

# Think Like a Production Engineer

Amazon S3 is much more than a file storage service—it is the foundation for backups, data lakes, static websites, application assets, CI/CD artifacts, logs, and analytics in AWS.

When designing production systems:

1. Use clear bucket naming conventions.
2. Organize data with prefixes.
3. Enable versioning and encryption.
4. Keep buckets private by default.
5. Use lifecycle policies to reduce storage costs.
6. Choose the appropriate storage class based on access patterns.

A well-designed S3 architecture is secure, scalable, cost-efficient, and easy to manage.

# End of Part 1
