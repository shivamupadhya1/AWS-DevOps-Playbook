# Amazon EBS (Elastic Block Store)

> **Volume 1 – Compute & Networking**
>
> Version: 1.0
>
> Author: Shivam Upadhyay

---

# Chapter Objective

After this chapter you should be able to:

- Explain EBS from a production perspective.
- Select the correct EBS volume type.
- Troubleshoot EBS issues.
- Answer senior DevOps interview questions.
- Design highly available storage architectures.

---

# 1. Business Problem

Imagine your Spring Boot application is deployed on an EC2 instance.

```
EC2
├── Application
├── Logs
├── Images
└── Database Files
```

One day someone accidentally stops the EC2 instance.

Question:

Will all the application data disappear?

AWS needed persistent storage that survives instance stop/start operations.

To solve this problem, AWS introduced **Elastic Block Store (EBS).**

---

# 2. What is Amazon EBS?

Amazon EBS is a persistent block storage service for EC2 instances.

It behaves like a physical hard disk attached to a server.

It stores:

- Operating System
- Application binaries
- Logs
- Configuration
- Databases
- Uploaded files

Unlike instance store volumes, EBS persists even if the EC2 instance is stopped.

---

# 3. Internal Working

```
Application

↓

Operating System

↓

File System

↓

EBS Volume

↓

AWS Storage Infrastructure
```

The operating system sees EBS as a normal disk (for example `/dev/xvda` or `/dev/nvme0n1`).

---

# 4. Volume Types

## gp3 (General Purpose SSD)

Best for:

- Web applications
- Jenkins
- SonarQube
- Spring Boot
- General workloads

Most commonly used in production.

---

## io2

Best for:

- Oracle
- SQL Server
- High-performance databases

Provides guaranteed IOPS.

---

## st1

Best for:

- Big data
- Sequential throughput
- Log processing

---

## sc1

Best for:

- Cold storage
- Infrequently accessed data

---

# 5. Request Flow

```
Application

↓

Operating System

↓

File System

↓

EBS Driver

↓

AWS Storage

↓

Disk Blocks
```

---

# 6. Production Architecture

```
Internet

↓

ALB

↓

EC2

↓

EBS

↓

Snapshots

↓

S3 (Managed by AWS)
```

---

# 7. Key Features

- Persistent Storage
- Encryption
- Snapshots
- Resize Volume
- High Availability within an Availability Zone
- Attach/Detach Volumes

---

# 8. Production Scenarios

## Scenario 1

Disk Full

Symptoms

```
Application crashes

↓

"No space left on device"
```

Investigation

```
df -h

↓

du -sh *

↓

Large Log Files

↓

Application Uploads

↓

Docker Images
```

Resolution

- Extend EBS volume.
- Extend the filesystem.
- Remove unnecessary files.

---

## Scenario 2

Application suddenly becomes slow.

CloudWatch

```
CPU = 20%
```

Memory

```
Normal
```

Question

What next?

Possible reason:

High Disk I/O.

Check

```
CloudWatch

↓

VolumeQueueLength

↓

ReadLatency

↓

WriteLatency

↓

IOPS
```

---

## Scenario 3

EC2 accidentally terminated.

Can data be recovered?

Answer

Only if:

- Snapshot exists
- Volume retained
- AMI available

---

# 9. Volume Expansion

Suppose

Current

```
50 GB
```

Application requires

```
100 GB
```

Steps

```
Modify Volume

↓

Wait

↓

Grow File System

↓

Verify
```

Linux Commands

```bash
lsblk

df -h

growpart

resize2fs

xfs_growfs
```

---

# 10. Encryption

EBS supports encryption using AWS KMS.

Encrypted volumes protect data:

- At Rest
- During Snapshot Creation
- During Snapshot Restore

---

# 11. AWS CLI

Create Volume

```bash
aws ec2 create-volume
```

Describe Volumes

```bash
aws ec2 describe-volumes
```

Attach Volume

```bash
aws ec2 attach-volume
```

Detach Volume

```bash
aws ec2 detach-volume
```

Modify Volume

```bash
aws ec2 modify-volume
```

---

# 12. Best Practices

- Use gp3 for most workloads.
- Enable encryption.
- Take regular snapshots.
- Monitor disk usage.
- Monitor IOPS.
- Delete unused volumes.
- Use lifecycle policies.

---

# 13. Common Mistakes

❌ Store backups only on one EBS volume.

Correct

↓

Snapshots

↓

Multiple AZ strategy

---

❌ Ignore disk usage alerts.

---

❌ Forget to resize the filesystem after increasing EBS size.

---

# 14. Interview Questions

---

## Question 1

Why did AWS introduce EBS?

### Perfect Answer

> EC2 instances need persistent storage for operating systems, applications, logs, and databases. Without EBS, stopping or replacing an instance could result in data loss. EBS provides durable block storage that can be attached, detached, resized, encrypted, and backed up using snapshots.

### Common Wrong Answer

"EBS is just AWS storage."

This answer is too generic.

---

## Question 2

What happens if you stop an EC2 instance?

### Perfect Answer

> By default, the root EBS volume remains attached and the data is preserved. When the instance starts again, the operating system, applications, and files are still available. If the instance is terminated and the root volume is configured to delete on termination, the volume will be removed.

### Interview Cross Question

What if the instance is terminated instead of stopped?

---

## Question 3

Difference between EBS and S3?

### Perfect Answer

| EBS | S3 |
|-----|----|
| Block Storage | Object Storage |
| Attached to EC2 | Accessed via API |
| Low latency | Massive scalability |
| Boot volume | File/object storage |

---

## Question 4

Your application reports

```
No space left on device
```

How do you troubleshoot?

### Perfect Answer

1. Check disk usage using `df -h`.
2. Identify large directories with `du -sh`.
3. Verify whether logs, uploads, or Docker images are consuming space.
4. If required, extend the EBS volume.
5. Extend the filesystem.
6. Verify the application.

---

## Question 5

Application becomes slow.

CPU

20%

Memory

40%

Where do you investigate?

### Perfect Answer

I would investigate disk performance because low CPU and memory indicate another bottleneck. I would review CloudWatch metrics such as VolumeQueueLength, ReadLatency, WriteLatency, and IOPS. On the instance I would use tools like `iostat`, `iotop` (if installed), and `vmstat` to identify heavy disk activity. If the workload has outgrown the current volume, I would consider increasing performance or changing the EBS volume type.

---

# 15. Amazon Cross Questions

### Question

Can one EBS volume be attached to two EC2 instances?

### Perfect Answer

Normally, no.

An EBS volume can be attached to only one EC2 instance at a time.

Exception:

Some io1/io2 Multi-Attach volumes support attachment to multiple instances in the same Availability Zone, but only for applications designed to handle shared block storage.

---

### Question

Can an EBS volume move automatically to another Availability Zone?

### Perfect Answer

No.

EBS volumes are scoped to a single Availability Zone. To use the data in another AZ, create a snapshot and restore it as a new EBS volume in the target AZ.

---

# 16. Hands-on Lab

Objective

Learn EBS management.

Steps

1. Launch an EC2 instance.
2. Create a 10 GB gp3 volume.
3. Attach it.
4. Format the disk.
5. Mount it.
6. Write sample files.
7. Create a snapshot.
8. Expand the volume to 20 GB.
9. Extend the filesystem.
10. Verify the new capacity.
11. Detach and reattach the volume.

---

# 17. One-Page Revision Sheet

Definition

- EBS is persistent block storage for EC2.

Remember

- AZ-specific
- Persistent
- Supports snapshots
- Supports encryption
- Supports resizing
- Boot volume for EC2

Troubleshooting Flow

```
Application Slow

↓

CloudWatch

↓

Disk Metrics

↓

df -h

↓

iostat

↓

Volume Type

↓

Expand Volume

↓

Verify
```

---

# Key Takeaways

Amazon EBS is the primary persistent storage service for EC2 instances. Understanding volume types, performance metrics, resizing, snapshots, encryption, and recovery strategies is essential for building reliable production systems. In interviews, always explain not only what EBS is, but also how you would monitor, troubleshoot, and recover from storage-related issues.
