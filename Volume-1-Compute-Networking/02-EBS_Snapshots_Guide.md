# AWS EBS Snapshots — Hands-on Lab & 4-Year DevOps Interview Revision Guide

## 1. What We Practiced

We used an EBS snapshot scenario to simulate a real production DevOps workflow:

- Creating a separate EBS data volume for application data
- Formatting and mounting the volume on EC2
- Simulating production application data
- Creating an EBS snapshot
- Understanding point-in-time recovery
- Deleting data and restoring it from a snapshot
- Understanding incremental snapshots
- Understanding what happens when an older snapshot is deleted
- Selecting the correct snapshot after a production incident
- Understanding RPO and recovery points
- Understanding crash consistency vs application consistency
- Discussing database backup considerations

The goal was not just to memorize AWS definitions, but to answer questions like a **4-year DevOps engineer** using production reasoning.

---

# 2. EBS Snapshot — Core Concept

An Amazon EBS snapshot is a point-in-time backup of an EBS volume.

A snapshot represents the data state of the volume at the time the snapshot was taken.

Typical recovery flow:

```text
EBS Volume
    |
    | Create Snapshot
    v
EBS Snapshot
    |
    | Create Volume from Snapshot
    v
New EBS Volume
    |
    | Attach
    v
EC2 Instance
    |
    | Mount
    v
Recovered Data
```

## Important terminology

Do not say that you "attach a snapshot."

A snapshot is a backup object.

The normal process is:

```text
Snapshot
   ↓
Create EBS volume from snapshot
   ↓
Attach EBS volume to EC2
   ↓
Mount filesystem
   ↓
Access recovered data
```

---

# 3. Hands-on Lab — Production EBS Backup & Disaster Recovery

## Scenario

You are a DevOps engineer responsible for a production EC2 server.

The server has an EBS data volume containing application data. Management wants a backup strategy so that if the volume is accidentally deleted or corrupted, the team can restore the data.

Your task is to create, test, and recover from an EBS snapshot.

---

## Step 1 — Create EC2

Use a small/free-tier-eligible instance available in your AWS account.

Recommended practice environment:

```text
OS              : Amazon Linux 2023
Instance        : Small/free-tier-eligible option
Region          : ap-south-1
Root volume     : Small volume
```

Avoid unnecessarily large resources to control cost.

---

# 4. Create a Separate Application/Data Volume

Go to:

```text
EC2
  → Elastic Block Store
  → Volumes
  → Create volume
```

Create a small `gp3` volume.

Important:

```text
Volume AZ = EC2 AZ
```

A newly created EBS volume must be in the appropriate Availability Zone to attach to the EC2 instance.

Attach the volume to the EC2 instance.

---

# 5. Identify the New Disk

Connect to EC2:

```bash
lsblk
```

Example:

```text
nvme0n1      10G
└─nvme0n1p1  10G /

nvme1n1       8G
```

The exact device name can vary.

---

# 6. Format the Volume

For an XFS filesystem:

```bash
sudo mkfs -t xfs /dev/nvme1n1
```

Create a mount point:

```bash
sudo mkdir /data
```

Mount:

```bash
sudo mount /dev/nvme1n1 /data
```

Verify:

```bash
df -h
```

---

# 7. Simulate Production Application Data

Create application data:

```bash
sudo mkdir -p /data/application
```

Example files:

```bash
sudo bash -c 'echo "Production Database Backup" > /data/application/database.txt'

sudo bash -c 'echo "Customer transaction data" > /data/application/transactions.txt'

sudo bash -c 'echo "Application configuration" > /data/application/config.txt'
```

Verify:

```bash
ls -lah /data/application
cat /data/application/database.txt
```

---

# 8. Create the EBS Snapshot

Go to:

```text
EC2
  → Elastic Block Store
  → Volumes
  → Select data volume
  → Actions
  → Create snapshot
```

Example description:

```text
prod-app-data-before-maintenance
```

Useful tags:

```text
Name        = prod-app-data-backup
Environment = production
Purpose     = disaster-recovery
```

Wait for the snapshot to reach its completed state.

---

# 9. Production Failure Simulation

Simulate accidental deletion of application data:

```bash
sudo rm -rf /data/application/*
```

Verify:

```bash
ls -lah /data/application
```

Imagine the application team reports:

> "Our production data disappeared. Restore it from the backup."

---

# 10. Restore From Snapshot

Go to:

```text
EC2
  → Snapshots
  → Select snapshot
  → Actions
  → Create volume from snapshot
```

Create the volume in the appropriate Availability Zone.

Attach the restored volume to the EC2 instance.

Check:

```bash
lsblk
```

Identify the restored device.

Check filesystem:

```bash
sudo blkid
```

Create a restore mount point:

```bash
sudo mkdir /restore
```

Mount the restored filesystem:

```bash
sudo mount /dev/nvme2n1 /restore
```

Verify:

```bash
ls -lah /restore/application
```

Expected files:

```text
database.txt
transactions.txt
config.txt
```

You have now performed an EBS disaster recovery exercise.

---

# 11. Snapshot Size — Important Interview Concept

## Question

You have a 500 GB EBS volume but only 120 GB of data is being used.

You create a snapshot.

Is the snapshot simply a 500 GB copy?

## Answer

Do not think of an EBS snapshot as a traditional full-file copy.

EBS snapshots use block-level storage and are incremental after the initial snapshot. AWS optimizes the storage by retaining the blocks required to represent the snapshots.

A strong interview answer is:

> "An EBS snapshot is a point-in-time block-level backup. Subsequent snapshots are incremental, so AWS does not need to store unchanged blocks repeatedly."

Be careful with simplistic statements such as:

> "The snapshot is exactly the amount of used filesystem data."

The actual snapshot storage behavior is based on blocks that need to be stored, not simply `df -h` used-space numbers.

---

# 12. Incremental Snapshots

Suppose:

```text
Snapshot-1
A B C D
```

Then block B changes and a new block E is created.

Snapshot-2 conceptually represents:

```text
A B' C D E
```

AWS can optimize storage by retaining the original unchanged blocks and storing the changed/new blocks.

Conceptually:

```text
Snapshot-1
A B C D
     \
      \
Snapshot-2
A B' C D E
```

The important interview statement:

> "A later snapshot represents the complete point-in-time state, while AWS optimizes the underlying storage by retaining only the blocks that are needed."

---

# 13. What Happens When an Older Snapshot Is Deleted?

## Interview Question

You have:

```text
Snapshot-1
Snapshot-2
Snapshot-3
```

If Snapshot-1 is deleted, will Snapshot-2 and Snapshot-3 become unusable?

## Answer

**No.**

AWS manages the underlying snapshot blocks and preserves blocks that are still required by later snapshots.

Conceptually:

```text
Before:

Snapshot-1 → A B C D
Snapshot-2 → B' E + required unchanged blocks

Delete Snapshot-1

Snapshot-2 → still restorable
```

You do not need to manually maintain a snapshot dependency chain.

A strong interview answer:

> "Deleting an older EBS snapshot does not make subsequent snapshots unusable. AWS manages the underlying blocks and retains blocks that are still required by other snapshots."

---

# 14. Important Terminology Correction

Do not say:

> "Snapshot-2 only contains the changed data."

Better:

> "Snapshot-2 represents the complete point-in-time state of the volume, while AWS optimizes the underlying snapshot storage by storing changed blocks and retaining blocks required from earlier snapshots."

This distinction prevents a common interview misunderstanding.

---

# 15. Snapshot Deletion — Production Scenario

## Scenario

You have:

```text
Monday    → Snapshot-1
Tuesday   → Snapshot-2
Wednesday → Snapshot-3
Thursday  → Snapshot-4
```

Management says:

> "We only need backups for the last two days. Delete Monday and Tuesday."

## Weak answer

> "I'll delete Snapshot-1 and Snapshot-2 because Snapshot-3 and Snapshot-4 will still work."

This is technically incomplete.

## Better 4-year DevOps answer

Before deletion, check:

### 1. Backup/retention policy

Confirm that the requested retention period is actually approved.

### 2. Snapshot status

Confirm that the remaining snapshots completed successfully.

### 3. Recovery coverage

Confirm that the remaining snapshots satisfy the application's recovery requirements and RPO.

### 4. DR requirements

Check whether copies exist in another region/account or whether another backup system has retention requirements.

### 5. Backup automation

Check whether a backup job, AWS Backup plan, lifecycle policy, or automation manages the snapshots.

### 6. Compliance

Check whether regulatory or organizational requirements require longer retention.

Strong interview answer:

> "Before deleting old production snapshots, I would verify the approved retention policy, confirm that the newer snapshots completed successfully, verify that they satisfy the application's RPO and recovery requirements, and check for DR, backup automation, or compliance requirements. Only then would I delete the old snapshots."

---

# 16. Production Database Recovery Scenario

## Scenario

Your application takes an EBS snapshot every night at 2 AM.

Snapshots:

```text
Sept 5  02:00 → Snapshot-A
Sept 6  02:00 → Snapshot-B
Sept 7  02:00 → Snapshot-C
Sept 8  02:00 → Snapshot-D
```

The database became corrupted on:

```text
Sept 7 at 5 PM
```

## Question

Which snapshot should you initially consider restoring?

## Answer

**Snapshot-C — Sept 7 at 2 AM.**

Why?

```text
Snapshot-C
Sept 7 — 02:00
      |
      | database changes
      |
      v
Sept 7 — 17:00
Database corruption
```

Snapshot-D was taken after the corruption and therefore may contain the corrupted state.

---

# 17. RPO — Recovery Point Objective

RPO answers:

> "How much data can the business afford to lose?"

In the example:

```text
Backup:       2:00 AM
Corruption:   5:00 PM
```

If you only have the 2 AM snapshot, you potentially have a large recovery gap.

You must ask the application/DB team:

> "Is losing the data/transactions generated between 2 AM and 5 PM acceptable?"

That is an RPO discussion.

---

# 18. Strong Interview Answer — Snapshot Selection

A good 4-year answer:

> "The corruption occurred at 5 PM on September 7, so I would initially select the September 7 2 AM snapshot because it is the latest known backup before the corruption. Before restoring, I would confirm with the application/DB team that the recovery point meets the application's RPO. I would also check whether database-native backups, transaction logs, WAL files, or binlogs are available so that we can potentially recover closer to the point of failure."

This is stronger than simply saying:

> "I'll restore Snapshot-C."

---

# 19. EBS Snapshot vs Database Consistency

This is one of the most important concepts from the discussion.

## Interview Question

A production database is actively writing data.

You take an EBS snapshot.

The snapshot status says:

```text
Completed
```

But after restoring it, the database has consistency problems.

How can that happen?

## Answer

An EBS snapshot operates at the **block-storage level**.

It does not inherently understand:

```text
Database transaction
Table relationships
Transaction boundaries
Application state
```

If a database is actively writing when the snapshot is taken, the resulting backup may be **crash-consistent** rather than fully **application-consistent**.

---

# 20. Crash-Consistent vs Application-Consistent

## Crash-consistent

The restored volume resembles what the system might look like after a sudden power failure.

The operating system/database may need to perform recovery.

```text
Database writing
      ↓
Sudden interruption
      ↓
Storage state
      ↓
Crash recovery
```

## Application-consistent

The application/database is coordinated before the backup so that its state is consistent from the application's perspective.

Conceptually:

```text
Pause/freeze or flush appropriately
        ↓
Create backup/snapshot
        ↓
Resume application
```

The exact mechanism depends on the application/database.

---

# 21. Why "Completed" Does Not Mean "Database-Consistent"

A snapshot status of:

```text
Completed
```

means the snapshot operation successfully completed from the EBS perspective.

It does **not automatically mean**:

```text
Database backup is transactionally consistent
```

This is a critical distinction.

Strong interview statement:

> "EBS snapshot completion confirms the storage snapshot completed, but it does not by itself guarantee application-level or transaction-level consistency for an actively writing database."

---

# 22. Better Production Database Backup Strategy

For important production databases, consider a combination of:

```text
Database-native backup
        +
Transaction logs / WAL / binlogs
        +
Storage snapshots where appropriate
        ↓
Point-in-time recovery
```

The exact design depends on the database and business RPO/RTO requirements.

A database-aware backup mechanism may provide better recovery semantics than relying only on an EBS snapshot.

---

# 23. Snapshot Recovery Flow

A production recovery flow can look like:

```text
Incident
   ↓
Identify failure time
   ↓
Find latest known-good backup
   ↓
Check RPO
   ↓
Confirm with application/DB team
   ↓
Create EBS volume from snapshot
   ↓
Attach to EC2
   ↓
Mount filesystem
   ↓
Validate data
   ↓
Recover application/database
   ↓
Perform application-level verification
```

---

# 24. Snapshot Interview Questions We Practiced

## Q1. 500 GB volume with only 120 GB used — what happens when you snapshot it?

Key points:

- EBS snapshots are block-level.
- Snapshot storage is not simply a traditional 500 GB file copy.
- Subsequent snapshots are incremental.
- Do not equate filesystem used space directly with snapshot storage consumption.

---

## Q2. What happens if you delete Snapshot-1 while Snapshot-2 and Snapshot-3 exist?

Answer:

- Snapshot-2 and Snapshot-3 remain usable.
- AWS manages underlying blocks.
- Blocks still required by later snapshots are retained.

---

## Q3. If Snapshot-1 is deleted, does AWS immediately copy all its data into Snapshot-2?

Answer:

- No.
- AWS does not need to immediately create a new full copy.
- AWS manages the underlying blocks and retains data required by remaining snapshots.

---

## Q4. What should you check before deleting old production snapshots?

Answer:

- Retention policy
- RPO/recovery requirements
- Snapshot completion status
- DR copies
- Backup automation/AWS Backup
- Compliance requirements
- Whether the snapshots are still required by the organization's recovery strategy

---

## Q5. Database corruption occurred at 5 PM. Snapshots run at 2 AM. Which snapshot do you initially consider?

Answer:

- The 2 AM snapshot from the same day, assuming it is known-good.
- Then investigate database-native backups and transaction logs for point-in-time recovery.

---

## Q6. What happens to changes after a snapshot?

Answer:

- They are not part of that already-created snapshot.
- A later snapshot can capture subsequent changed blocks.
- The original snapshot continues to represent its own recovery point.

---

## Q7. Can you attach an EBS snapshot directly to EC2?

Answer:

No.

Correct flow:

```text
Snapshot
   ↓
Create EBS volume
   ↓
Attach volume
   ↓
Mount volume
```

---

## Q8. Snapshot status says "Completed." Can the database still have consistency problems?

Answer:

Yes.

Reason:

- EBS works at block-storage level.
- The database may have been actively writing.
- The snapshot can be crash-consistent rather than application-consistent.
- Database-aware backup/recovery mechanisms may be needed.

---

# 25. Common Interview Mistakes

### Mistake 1

> "Snapshot-2 only contains changed files."

Better:

> "Snapshot-2 represents the complete point-in-time state, while AWS optimizes the underlying storage using incremental block storage."

---

### Mistake 2

> "If I delete Snapshot-1, Snapshot-2 loses Snapshot-1's data."

Wrong.

AWS preserves blocks required by remaining snapshots.

---

### Mistake 3

> "Completed snapshot means the database backup is consistent."

Not necessarily.

Snapshot completion is not the same as application consistency.

---

### Mistake 4

> "I'll delete old snapshots because newer snapshots exist."

Incomplete.

First verify:

```text
Retention
RPO
DR
Compliance
Backup policies
Recovery requirements
```

---

### Mistake 5

> "I'll restore the latest snapshot."

Not necessarily.

If the latest snapshot was taken **after corruption**, restoring it may restore corrupted data.

Always determine:

```text
Failure time
+
Backup creation time
+
Known-good recovery point
```

---

# 26. Production Thinking: RPO and RTO

Two important interview terms:

## RPO — Recovery Point Objective

How much data loss is acceptable?

Example:

```text
RPO = 1 hour
```

The business expects to recover to a point no more than approximately one hour before the incident, depending on the backup architecture.

## RTO — Recovery Time Objective

How quickly must the service be restored?

Example:

```text
RTO = 30 minutes
```

The recovery solution should be designed to restore service within that target.

---

# 27. Example Architecture

A more mature backup architecture might look like:

```text
                 Production EC2
                       |
                EBS Data Volume
                       |
             +---------+---------+
             |                   |
       EBS Snapshots       DB-native backup
             |                   |
             |            WAL/binlog/etc.
             |                   |
             +---------+---------+
                       |
                  DR Strategy
                       |
          Cross-region / backup copy
```

The correct architecture depends on:

- RPO
- RTO
- Data criticality
- Database technology
- Compliance
- Cost
- Recovery requirements

---

# 28. What You Should Be Able to Explain in an Interview

At a 4-year level, you should be able to explain:

### Basic

- What is an EBS snapshot?
- Why are snapshots useful?
- How do you create one?
- How do you restore one?

### Intermediate

- Incremental snapshots
- Point-in-time recovery
- Snapshot deletion
- Snapshot retention
- Snapshot storage/cost considerations

### Advanced

- Crash consistency
- Application consistency
- Database backup strategy
- RPO/RTO
- Disaster recovery
- Cross-region recovery
- Backup automation
- Production retention policies

---

# 29. Quick Revision Cheat Sheet

```text
EBS Snapshot
    ↓
Point-in-time block-level backup

Restore
    ↓
Snapshot → EBS Volume → EC2 → Mount

Subsequent snapshots
    ↓
Incremental storage optimization

Delete older snapshot
    ↓
Later snapshots remain usable
    ↓
AWS manages required blocks

Production restore
    ↓
Find failure time
    ↓
Find latest known-good backup
    ↓
Check RPO
    ↓
Restore
    ↓
Validate

Database
    ↓
EBS snapshot ≠ automatically application-consistent
    ↓
Consider database-aware backup + logs
```

---

# 30. Final Interview Model Answer

If an interviewer asks:

> "Explain how you would use EBS snapshots for production backup and recovery."

A strong answer is:

> "I would use EBS snapshots as part of the backup strategy for EC2 EBS volumes. I would define the retention policy based on business RPO/RTO requirements and automate snapshot creation and cleanup rather than managing them manually.
>
> In a failure, I would identify the failure time and select the latest known-good snapshot before the incident. I would create a new EBS volume from that snapshot, attach it to the appropriate EC2 instance, mount it, and validate the recovered data before bringing the application back.
>
> For databases, I would not assume that a successfully completed EBS snapshot is automatically application-consistent. I would coordinate with the database/application team and consider database-native backups and transaction logs for point-in-time recovery.
>
> I would also consider encryption, retention, cross-region DR, monitoring, and backup validation as part of a production-grade solution."

---

# 31. Labs to Do Next

Continue practicing snapshots in this order:

```text
LAB 1
Create snapshot
→ Restore snapshot
→ Verify data

LAB 2
Delete application data
→ Recover from snapshot

LAB 3
Terminate EC2
→ Launch new EC2
→ Restore EBS volume
→ Recover data

LAB 4
Create multiple snapshots
→ Modify data between snapshots
→ Delete older snapshots
→ Restore newer snapshot

LAB 5
Database consistency
→ Understand crash vs application consistency

LAB 6
Snapshot retention
→ Automate backup/cleanup

LAB 7
Cross-region snapshot copy
→ Restore in another region

LAB 8
Production backup architecture
→ RPO
→ RTO
→ DR
→ Encryption
→ IAM
→ Monitoring
```

## Interview mindset

Don't answer only:

> "AWS supports snapshots."

Answer in terms of:

```text
Problem
   ↓
Business requirement
   ↓
RPO/RTO
   ↓
Backup strategy
   ↓
Failure scenario
   ↓
Recovery procedure
   ↓
Validation
   ↓
Automation
   ↓
Cost + security + DR
```

That is the level of thinking you should aim for in a **4-year DevOps interview**.
