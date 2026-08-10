# Amazon Machine Image (AMI)

> **AWS DevOps Playbook**
>
> Volume 1 – Compute & Networking
>
> Chapter 03

---

# Chapter Objective

After completing this chapter you should be able to:

- Explain AMI from a production perspective.
- Understand how EC2 boots internally.
- Create reusable golden images.
- Design Auto Scaling architectures using AMIs.
- Answer senior DevOps interview questions confidently.

---

# 1. Business Problem

Imagine your company has 100 production EC2 instances.

Each server requires:

- Ubuntu
- Java 21
- Tomcat
- CloudWatch Agent
- Security Agent
- Monitoring Scripts
- Company Certificates
- Application Configuration

Now imagine one EC2 instance crashes.

Should an engineer manually install everything again?

No.

It could take hours and increase the chance of configuration differences.

AWS solved this problem by introducing **Amazon Machine Image (AMI)**.

---

# 2. What is an AMI?

An Amazon Machine Image (AMI) is a **pre-configured template** used to launch EC2 instances.

It contains:

- Operating System
- Installed Software
- Packages
- Security Updates
- Configuration Files
- Application Code (optional)

Think of it as a **master image** or **golden template** for creating identical EC2 instances.

---

# 3. Internal Working

```
Golden EC2

↓

Install Software

↓

Configure Server

↓

Create AMI

↓

AMI Stored

↓

Launch Multiple EC2 Instances

↓

All Instances Are Identical
```

When a new EC2 instance is launched from an AMI, AWS creates a new root EBS volume from the AMI snapshot and boots the operating system from it.

---

# 4. What Does an AMI Contain?

An AMI includes:

- Root EBS Snapshot
- Operating System
- Boot Configuration
- Block Device Mapping
- Launch Permissions
- Metadata

An AMI does **not** include:

- Running processes
- Current RAM contents
- Temporary CPU state
- Active network connections

---

# 5. Production Architecture

```
Golden Server

↓

Install

Java

↓

Tomcat

↓

CloudWatch Agent

↓

Security Agent

↓

Application

↓

Create AMI

↓

Auto Scaling Group

↓

Launch Hundreds of EC2
```

This is the standard approach used in production environments.

---

# 6. Request Flow

```
User

↓

Launch EC2

↓

Select AMI

↓

Create Root EBS Volume

↓

Hypervisor

↓

Boot Operating System

↓

Application Starts
```

---

# 7. Types of AMIs

## AWS Managed AMI

Provided by AWS.

Examples:

- Ubuntu
- Amazon Linux
- Windows

---

## Custom AMI

Created by your organization.

Contains:

- Company Software
- Monitoring
- Security Tools
- Application Configuration

---

## Marketplace AMI

Published by third-party vendors.

Examples:

- Jenkins
- GitLab
- SonarQube
- Fortinet Firewall

---

# 8. Golden AMI

One of the most common interview topics.

A Golden AMI is a pre-approved production image containing everything required to run applications securely.

Example:

```
Ubuntu

+

Java

+

CloudWatch Agent

+

Security Agent

+

Company Certificates

+

Hardening

=

Golden AMI
```

Benefits:

- Faster deployment
- Standard configuration
- Reduced manual work
- Better security
- Consistent servers

---

# 9. Production Scenarios

## Scenario 1

Auto Scaling launches unhealthy servers.

Possible Cause:

Old AMI.

Example:

AMI contains

```
Java 11
```

Application requires

```
Java 21
```

Solution:

- Build a new AMI.
- Update Launch Template.
- Perform Rolling Update.

---

## Scenario 2

Security team reports that production servers are missing a critical patch.

Question:

Should you patch every EC2 manually?

No.

Correct approach:

- Update the Golden AMI.
- Test it.
- Update Launch Template.
- Replace existing instances gradually.

---

## Scenario 3

A new EC2 launches successfully but the application does not start.

Possible Causes:

- Startup script failure
- User Data issue
- Application service disabled
- Incorrect AMI version
- Missing dependencies

---

# 10. Real Production Story

A company maintained one manually configured server.

Every new engineer configured servers differently.

Results:

- Different Java versions
- Different package versions
- Inconsistent monitoring
- Deployment failures

After implementing Golden AMIs:

- Every server became identical.
- Auto Scaling became reliable.
- Deployments became predictable.
- Troubleshooting became easier.

---

# 11. AWS CLI

List AMIs

```bash
aws ec2 describe-images --owners self
```

Create AMI

```bash
aws ec2 create-image \
  --instance-id i-123456789 \
  --name "springboot-prod-v1"
```

Delete AMI

```bash
aws ec2 deregister-image \
  --image-id ami-xxxxxxxx
```

---

# 12. Best Practices

- Maintain Golden AMIs.
- Patch regularly.
- Version your AMIs.
- Remove unused AMIs.
- Use Launch Templates.
- Encrypt EBS volumes.
- Automate image creation using Image Builder or Packer.

---

# 13. Common Mistakes

❌ Creating one AMI and using it forever.

Correct:

Regularly patch and create new versions.

---

❌ Launching production servers from public AMIs.

Correct:

Use organization-approved Golden AMIs.

---

❌ Forgetting to update Auto Scaling Launch Templates.

---

# 14. Interview Questions

---

## Question 1

Why did AWS introduce AMIs?

### Perfect Answer

AWS introduced AMIs to eliminate repetitive server configuration. Instead of installing the operating system, software, security tools, and application dependencies every time, organizations can create a reusable machine image and launch identical EC2 instances quickly. This ensures consistency, faster deployments, and easier scaling.

---

## Question 2

Difference between AMI and Snapshot?

### Perfect Answer

| AMI | Snapshot |
|------|----------|
| Launches EC2 | Backup of an EBS volume |
| Contains boot configuration | Contains only disk data |
| Used by Auto Scaling | Used for restore |
| Includes metadata | No launch metadata |

---

## Question 3

What happens internally when an EC2 instance is launched from an AMI?

### Perfect Answer

1. User selects an AMI.
2. AWS creates a new EBS root volume from the AMI snapshot.
3. The selected instance type is provisioned.
4. The operating system boots.
5. User Data scripts (if any) run.
6. The instance becomes available.

---

## Question 4

Why is AMI important in Auto Scaling?

### Perfect Answer

Auto Scaling launches new EC2 instances automatically during scale-out events. To ensure every new instance has the same operating system, software versions, security patches, monitoring agents, and application configuration, it uses an AMI defined in the Launch Template.

---

## Question 5

Your Auto Scaling Group keeps launching unhealthy instances.

How would you investigate?

### Perfect Answer

1. Verify the AMI version.
2. Check User Data execution.
3. Confirm the application service started successfully.
4. Review system logs and CloudWatch logs.
5. Verify security agents and required software are present.
6. Launch one EC2 manually from the same AMI for testing.

---

# 15. Amazon Cross Questions

### Question

Can you modify an existing AMI?

### Perfect Answer

No.

An AMI is immutable. If changes are required, launch an EC2 instance from the AMI, apply updates, and create a new AMI version.

---

### Question

Should applications be installed manually after every Auto Scaling launch?

### Perfect Answer

No.

Production environments typically install everything before creating a Golden AMI. Auto Scaling launches instances from that AMI so that every server is identical.

---

### Question

How do organizations keep Golden AMIs updated?

### Perfect Answer

Most organizations use automated image pipelines with AWS Image Builder or HashiCorp Packer. New patches are applied, images are tested, versioned, and then promoted for production use.

---

# 16. Hands-on Lab

Objective:

Create and use a custom AMI.

Steps:

1. Launch Ubuntu EC2.
2. Install Java.
3. Install Nginx.
4. Create a sample HTML page.
5. Install CloudWatch Agent.
6. Create a Custom AMI.
7. Launch another EC2 from the AMI.
8. Verify all software is already installed.
9. Update the server.
10. Create AMI v2.
11. Compare both versions.

---

# 17. AMI vs Snapshot vs EBS

| Feature | AMI | Snapshot | EBS |
|----------|-----|----------|-----|
| Boot EC2 | ✅ | ❌ | ❌ |
| Backup | Partial | ✅ | ❌ |
| Block Storage | ❌ | ❌ | ✅ |
| Launch Configuration | ✅ | ❌ | ❌ |
| Used in Auto Scaling | ✅ | ❌ | Indirectly |

---

# 18. Revision Sheet

Definition

- AMI is a reusable machine image used to launch EC2 instances.

Remember

- Built from snapshots
- Used in Launch Templates
- Required by Auto Scaling
- Immutable
- Versioned
- Supports Golden Images

Troubleshooting Flow

```
Auto Scaling

↓

Launch Failed

↓

AMI

↓

Launch Template

↓

User Data

↓

Application

↓

Health Check
```

---

# 19. Think Like a Production Engineer

Don't think:

> "AMI is a template."

Think:

> "AMI guarantees every server in production is identical."

Whenever someone asks:

"How do you ensure consistency across hundreds of EC2 instances?"

Your first answer should be:

**Golden AMIs + Launch Templates + Auto Scaling.**

---

# Key Takeaways

Amazon AMIs are the foundation of repeatable infrastructure in AWS. They enable standardized server deployments, support Auto Scaling, reduce manual configuration, and improve operational consistency. In production, Golden AMIs combined with automated image pipelines ensure every new instance is secure, patched, and identical to the approved baseline.

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

EBS Snapshots are incremental, point-in-time backups of EBS volumes stored in AWS-managed S3. They form the foundation of backup and disaster recovery strategies for EC2 workloads. A strong DevOps engineer understands not only how to create snapshots, but also how to automate them, restore them, test recoveries, and integrate them into production disaster recovery plans.
