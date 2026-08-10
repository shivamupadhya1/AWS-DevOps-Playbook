# Amazon EBS Snapshots

> **AWS DevOps Playbook**
>
> Volume 1 – Compute & Networking
>
> Chapter 04

---

# Chapter Objective

After completing this chapter you should be able to:

- Explain how EBS Snapshots work internally.
- Explain incremental snapshots.
- Design disaster recovery strategies.
- Restore production environments.
- Answer advanced DevOps interview questions.

---

# 1. Business Problem

Imagine your production database runs on an EC2 instance.

```
EC2

↓

100 GB EBS Volume

↓

MySQL Database
```

One day

```
rm -rf /
```

or

Database corruption.

Question

How do you recover?

AWS introduced **Snapshots** to solve backup and disaster recovery problems.

---

# 2. What is an EBS Snapshot?

An EBS Snapshot is a **point-in-time backup** of an EBS volume.

AWS stores snapshots in **Amazon S3 (managed internally)**.

Important:

You **cannot** browse that S3 bucket.

AWS manages it completely.

---

# 3. Internal Working

```
EC2

↓

EBS Volume

↓

Snapshot

↓

Stored in AWS Managed S3

↓

Restore

↓

New EBS Volume

↓

Attach to EC2
```

Snapshots are **not attached** to EC2.

They are backups used to create new EBS volumes.

---

# 4. Full vs Incremental Snapshots

This is one of the most frequently asked interview topics.

Suppose

Day 1

```
100 GB Data
```

Create Snapshot 1

AWS stores

```
100 GB
```

---

Day 2

Only

```
5 GB Changed
```

Question

Does AWS store another 100 GB?

No.

AWS stores only

```
5 GB
```

This is called

**Incremental Snapshot.**

---

Visualization

```
Snapshot 1

100 GB

↓

Snapshot 2

+5 GB

↓

Snapshot 3

+2 GB

↓

Snapshot 4

+1 GB
```

This saves storage costs.

---

# 5. Request Flow

```
EBS Volume

↓

Create Snapshot

↓

Changed Blocks

↓

AWS Managed S3

↓

Snapshot Created
```

---

# 6. Production Architecture

```
Application

↓

EC2

↓

EBS

↓

Snapshot

↓

Cross Region Copy (Optional)

↓

Disaster Recovery
```

---

# 7. Production Scenarios

## Scenario 1

Developer accidentally deletes production files.

Solution

```
Snapshot

↓

Create New Volume

↓

Attach

↓

Recover
```

---

## Scenario 2

Availability Zone failure.

Question

Can you attach the same EBS volume to another AZ?

No.

Correct Solution

```
Snapshot

↓

Create New Volume

↓

Target AZ

↓

Attach

↓

Recover
```

---

## Scenario 3

Production server terminated.

Question

How do you recover?

Perfect Answer

1. Launch new EC2.
2. Restore EBS volume from Snapshot.
3. Attach restored volume.
4. Mount filesystem.
5. Verify application.
6. Update DNS/ALB if required.

---

# 8. Real Production Story

A developer accidentally deleted customer-uploaded documents.

Fortunately

Snapshots were taken every night.

The team

- Restored the snapshot.
- Created a new EBS volume.
- Mounted it.
- Copied only the required files.
- Production recovered within one hour.

Without snapshots

The data would have been permanently lost.

---

# 9. Snapshot Lifecycle

```
EBS

↓

Daily Snapshot

↓

Weekly Snapshot

↓

Monthly Snapshot

↓

Delete Old Snapshots
```

Normally automated using

Amazon Data Lifecycle Manager (DLM).

---

# 10. AWS CLI

Create Snapshot

```bash
aws ec2 create-snapshot \
  --volume-id vol-xxxxxxxx
```

Describe Snapshots

```bash
aws ec2 describe-snapshots
```

Copy Snapshot

```bash
aws ec2 copy-snapshot
```

Delete Snapshot

```bash
aws ec2 delete-snapshot \
  --snapshot-id snap-xxxxxxxx
```

---

# 11. Best Practices

- Automate snapshots.
- Encrypt snapshots.
- Copy critical snapshots to another Region.
- Test restore procedures.
- Apply lifecycle policies.
- Tag snapshots.

---

# 12. Common Mistakes

❌ Never testing restores.

A backup that cannot be restored is not a reliable backup.

---

❌ Keeping snapshots forever.

Storage cost increases unnecessarily.

---

❌ Assuming snapshots are stored inside EC2.

They are stored in AWS-managed S3.

---

# 13. Interview Questions

---

## Question 1

Why did AWS introduce Snapshots?

### Perfect Answer

Snapshots provide point-in-time backups of EBS volumes. They allow organizations to recover from accidental deletion, corruption, hardware failure, or disaster without manually copying files.

---

## Question 2

Where are Snapshots stored?

### Perfect Answer

Snapshots are stored in Amazon S3 managed internally by AWS. Customers do not have direct access to this bucket.

---

## Question 3

Explain Incremental Snapshots.

### Perfect Answer

The first snapshot copies all used blocks of an EBS volume. Every subsequent snapshot stores only the blocks that changed since the previous snapshot. AWS maintains the dependency chain internally while presenting each snapshot as a complete recovery point.

---

## Question 4

Can you restore a Snapshot directly to an EC2 instance?

### Perfect Answer

No.

A snapshot must first be restored as a new EBS volume. That volume is then attached to an EC2 instance.

---

## Question 5

Can a Snapshot be copied to another Region?

### Perfect Answer

Yes.

Snapshots can be copied across Regions for disaster recovery, migration, or compliance.

---

# 14. Amazon Cross Questions

### Question

Can you delete Snapshot 1 if Snapshot 2 exists?

### Perfect Answer

Yes.

AWS tracks changed blocks internally. Deleting an older snapshot does not remove blocks that are still required by newer snapshots.

---

### Question

Are Snapshots crash-consistent or application-consistent?

### Perfect Answer

By default, EBS snapshots are crash-consistent. For application-consistent backups (such as databases), you should quiesce the application or use mechanisms like AWS Systems Manager or database-native backup procedures before taking the snapshot.

---

### Question

Do Snapshots back up free space?

### Perfect Answer

No.

Snapshots store only the used blocks of the EBS volume, making them storage-efficient.

---

# 15. Hands-on Lab

Objective

Understand snapshot creation and recovery.

Steps

1. Launch EC2.
2. Create a 10 GB EBS volume.
3. Mount it.
4. Create sample files.
5. Create Snapshot.
6. Delete the files.
7. Restore Snapshot.
8. Create a new EBS volume.
9. Attach it.
10. Verify files are restored.
11. Copy Snapshot to another Region (optional).

---

# 16. Snapshot vs AMI vs EBS

| Feature | Snapshot | AMI | EBS |
|----------|----------|-----|-----|
| Backup | ✅ | Partial | ❌ |
| Launch EC2 | ❌ | ✅ | ❌ |
| Block Storage | ❌ | ❌ | ✅ |
| Restore Volume | ✅ | ❌ | ❌ |
| Used by Auto Scaling | ❌ | ✅ | Indirectly |

---

# 17. Revision Sheet

Remember

- Point-in-time backup
- Stored in AWS-managed S3
- Incremental
- AZ independent
- Can create new EBS volumes
- Supports cross-Region copy

Recovery Flow

```
Volume Failure

↓

Snapshot

↓

Create Volume

↓

Attach

↓

Mount

↓

Application
```

---

# 18. Think Like a Production Engineer

Don't think:

> "Snapshot is just a backup."

Think:

> "Snapshot is my disaster recovery mechanism."

Whenever an interviewer asks

> "Your production EBS volume is corrupted."

Your answer should immediately become:

```
Identify Snapshot

↓

Create New Volume

↓

Attach

↓

Validate

↓

Bring Service Back

↓

Perform RCA
```

---

# Key Takeaways

EBS Snapshots are incremental, point-in-time backups of EBS volumes stored in AWS-managed S3. They form the foundation of backup and disaster recovery strategies for EC2 workloads. A strong DevOps engineer understands not only how to create snapshots, but also how to automate them, restore them, test recoveries, and integrate 
