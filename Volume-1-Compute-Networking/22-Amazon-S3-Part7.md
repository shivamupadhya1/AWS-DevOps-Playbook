# Amazon S3

> AWS DevOps Playbook
>
> Volume 1 – Storage
>
> Chapter 22
>
> Amazon S3 – Part 7 (AWS CLI, DevOps, Monitoring & Real Production Scenarios)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Work with Amazon S3 using AWS CLI
- Understand S3 in CI/CD pipelines
- Learn S3 Logging
- Understand CloudWatch Monitoring
- Learn S3 Storage Lens
- Understand Inventory Reports
- Learn Batch Operations
- Learn Cost Optimization
- Design Production Architectures
- Prepare for DevOps Interviews

---

# 175. Amazon S3 and DevOps

One of the most common AWS services used in DevOps is Amazon S3.

It is used for:

- Jenkins Artifacts
- Terraform Remote State
- CloudFormation Templates
- Application Backups
- Log Storage
- Static Websites
- Docker Image Backups
- Lambda Deployment Packages

Example

```
Developer

↓

Jenkins

↓

Build

↓

Artifact

↓

Amazon S3
```

---

# 176. S3 in a CI/CD Pipeline

Typical production pipeline:

```
GitHub

↓

Jenkins

↓

Build

↓

WAR/JAR

↓

Amazon S3

↓

Deployment

↓

EC2 / ECS / EKS
```

Advantages:

- Central artifact storage
- Easy rollback
- Version history
- Highly durable

---

# 177. AWS CLI Basics

List buckets

```bash
aws s3 ls
```

Example Output

```
company-backups

jenkins-artifacts

terraform-state
```

---

# 178. Create Bucket

```bash
aws s3 mb s3://company-backup-bucket
```

Result

```
Bucket Created
```

---

# 179. Upload File

```bash
aws s3 cp app.jar s3://company-artifacts/
```

Flow

```
Local File

↓

Upload

↓

S3 Bucket
```

---

# 180. Download File

```bash
aws s3 cp s3://company-artifacts/app.jar .
```

Downloads the object to the current directory.

---

# 181. Upload Folder

```bash
aws s3 cp ./build s3://company-builds/ --recursive
```

Useful for:

- Website assets
- Build outputs
- Reports
- Images

---

# 182. Synchronize Folder

```bash
aws s3 sync ./website s3://company-website
```

Only changed files are uploaded.

This is much faster than uploading everything.

---

# 183. Download Entire Bucket

```bash
aws s3 sync s3://company-backup ./backup
```

Downloads all objects while preserving the directory structure.

---

# 184. Delete Object

```bash
aws s3 rm s3://company-artifacts/app.jar
```

Deletes the specified object.

---

# 185. Delete Folder

```bash
aws s3 rm s3://company-artifacts --recursive
```

Deletes every object inside the bucket.

---

# 186. Copy Between Buckets

```bash
aws s3 cp s3://bucket1/app.jar s3://bucket2/
```

Common Use Cases:

- Backup
- Migration
- Testing

---

# 187. Sync Between Buckets

```bash
aws s3 sync s3://bucket1 s3://bucket2
```

Useful for:

- Disaster Recovery
- Cross-account migration
- Backup

---

# 188. S3 Logging

Amazon S3 supports:

### Server Access Logging

Logs requests made to a bucket.

Example

```
User

↓

GET Object

↓

Logged
```

Information includes:

- Requester
- Time
- Bucket
- Operation
- Status code

---

# 189. CloudTrail vs S3 Access Logs

| S3 Server Access Logs | AWS CloudTrail |
|-----------------------|----------------|
| Object access requests | API activity |
| Bucket level | AWS account level |
| Used for traffic analysis | Used for auditing API calls |

Example:

CloudTrail records:

```
DeleteBucket

PutBucketPolicy

CreateBucket
```

S3 Access Logs record:

```
GET Object

PUT Object

DELETE Object
```

---

# 190. Amazon CloudWatch Metrics

Amazon S3 publishes metrics to CloudWatch.

Examples:

- Bucket Size
- Number of Objects

With Request Metrics enabled, you can also monitor:

- GET Requests
- PUT Requests
- Errors
- Latency

---

# 191. S3 Storage Lens

Storage Lens provides organization-wide visibility into S3 usage.

It shows:

- Storage trends
- Object counts
- Cost optimization opportunities
- Encryption status
- Versioning adoption

Example Dashboard

```
Buckets

↓

Storage Lens

↓

Insights

↓

Recommendations
```

---

# 192. S3 Inventory

Inventory creates reports of bucket contents.

Example Report

```
Bucket

↓

Inventory

↓

CSV

↓

Millions of Objects
```

Useful for:

- Audits
- Compliance
- Reporting
- Migration

---

# 193. Batch Operations

Suppose:

10 million objects

Need to:

- Add Tags
- Change ACL
- Copy Objects
- Invoke Lambda
- Restore Glacier Objects

Doing this manually is impossible.

Use:

```
S3 Batch Operations
```

---

# 194. Batch Operations Workflow

```
Inventory Report

↓

Batch Job

↓

Millions of Objects

↓

Operation

↓

Completed
```

---

# 195. Cost Optimization

Production Checklist

✔ Lifecycle Rules

✔ Intelligent Tiering

✔ Compression

✔ Delete Unused Objects

✔ Monitor Storage

✔ Remove Incomplete Multipart Uploads

✔ Use S3 Storage Lens

✔ Enable Bucket Keys for SSE-KMS

---

# 196. Production Architecture

```
Developer

↓

GitHub

↓

Jenkins

↓

Build

↓

Upload

↓

Amazon S3

↓

CloudFront

↓

Users
```

---

# 197. Backup Architecture

```
Production Bucket

↓

Versioning

↓

CRR

↓

Backup Region

↓

Deep Archive
```

---

# 198. Image Processing Architecture

```
User Upload

↓

Amazon S3

↓

Event Notification

↓

Lambda

↓

Resize

↓

Thumbnail Bucket
```

---

# 199. Static Website Architecture

```
Users

↓

CloudFront

↓

Amazon S3

↓

HTML

↓

CSS

↓

JavaScript
```

---

# 200. Enterprise Logging Architecture

```
Application

↓

Logs

↓

Amazon S3

↓

Lifecycle

↓

Glacier

↓

Deep Archive
```

---

# 201. Enterprise Data Lake

```
Applications

↓

Amazon S3

↓

Data Lake

↓

Athena

↓

Glue

↓

EMR

↓

QuickSight
```

Amazon S3 is commonly used as the storage layer for AWS analytics services.

---

# 202. Best Practices

✔ Enable Versioning.

✔ Enable Encryption.

✔ Enable Block Public Access.

✔ Use IAM Roles.

✔ Enable Lifecycle Rules.

✔ Enable Logging.

✔ Monitor Storage.

✔ Review Bucket Policies.

✔ Use CloudFront for public content.

✔ Use Pre-Signed URLs instead of public buckets.

---

# 203. Common Production Mistakes

❌ Public bucket containing confidential data.

❌ No encryption.

❌ No Versioning.

❌ No Lifecycle Rules.

❌ No Disaster Recovery.

❌ No Monitoring.

❌ No Backup.

❌ No IAM least privilege.

---

# 204. Real Interview Scenario 1

### Question

How would you design storage for Jenkins artifacts?

### Answer

```
Jenkins

↓

Build

↓

Amazon S3

↓

Versioning Enabled

↓

Lifecycle

↓

Standard-IA

↓

Glacier

↓

Delete After 1 Year
```

---

# 205. Real Interview Scenario 2

### Question

How would you store application logs?

### Answer

```
Application

↓

Amazon S3

↓

Lifecycle

↓

Glacier

↓

Deep Archive
```

Enable:

- Encryption
- Versioning (if required)
- Lifecycle Rules
- Access Logging

---

# 206. Real Interview Scenario 3

### Question

How would you securely share invoices?

### Answer

Store invoices in a private bucket and generate **Pre-Signed URLs** with short expiration times.

---

# 207. Real Interview Scenario 4

### Question

How would you design Disaster Recovery for S3?

### Answer

```
Primary Bucket

↓

Versioning

↓

Cross Region Replication

↓

Backup Bucket

↓

Lifecycle

↓

Deep Archive
```

---

# 208. Frequently Asked Interview Questions

## Question 48

Can Amazon S3 store an operating system?

### Answer

No.

Amazon S3 is an object storage service, not a block storage service.

---

## Question 49

What is the maximum object size in Amazon S3?

### Answer

**5 TB**

---

## Question 50

When should Multipart Upload be used?

### Answer

Recommended for objects larger than **5 GB**.

---

## Question 51

Can an EC2 instance mount an S3 bucket as a normal disk?

### Answer

Not natively.

S3 is object storage.

Tools like **Mountpoint for Amazon S3** or **s3fs** can provide file-like access, but they have different characteristics from traditional file systems.

---

## Question 52

What is the durability of Amazon S3?

### Answer

Amazon S3 is designed for **99.999999999% (11 nines)** durability.

---

## Question 53

What is the availability of S3 Standard?

### Answer

Amazon S3 Standard is designed for **99.99% availability**.

---

## Question 54

Can S3 trigger a Lambda function?

### Answer

Yes.

S3 Event Notifications can invoke AWS Lambda.

---

## Question 55

Can multiple EC2 instances read the same S3 object?

### Answer

Yes.

Multiple clients can read the same object simultaneously.

---

## Question 56

Can S3 replace Amazon EBS?

### Answer

No.

S3 provides object storage, while Amazon EBS provides block storage for EC2 instances.

---

## Question 57

Can Amazon S3 be used as a database?

### Answer

No.

S3 stores objects and is not a relational or NoSQL database.

---

# 209. Hands-on Labs

## Lab 31

Upload files using the AWS CLI.

---

## Lab 32

Synchronize a website to an S3 bucket using:

```bash
aws s3 sync
```

---

## Lab 33

Configure Server Access Logging.

---

## Lab 34

Generate an S3 Inventory report.

---

## Lab 35

Create a Batch Operations job.

---

## Lab 36

Design a secure production S3 architecture for a banking application.

---

# 210. Complete Amazon S3 Mind Map

```
Amazon S3

│

├── Buckets

├── Objects

├── Storage Classes

├── Versioning

├── Lifecycle

├── Replication

├── Encryption

├── IAM

├── Bucket Policy

├── Block Public Access

├── Multipart Upload

├── Pre-Signed URLs

├── Event Notifications

├── Static Website

├── CloudFront

├── Object Lock

├── Access Points

├── Multi-Region Access Points

├── Inventory

├── Storage Lens

├── Batch Operations

├── Monitoring

└── AWS CLI
```

---

# Final S3 Revision

Remember these interview keywords:

- Object Storage
- Unlimited Storage
- 5 TB Maximum Object Size
- 11 Nines Durability
- Multi-AZ (except One Zone-IA)
- Versioning
- Lifecycle
- Replication
- Encryption
- Multipart Upload
- Pre-Signed URL
- Event Notifications
- Static Website Hosting
- CloudFront
- Object Lock
- Access Points
- Storage Lens
- Inventory
- Batch Operations
- AWS CLI
- Disaster Recovery

---

# Think Like a Senior DevOps Engineer

Amazon S3 is far more than a place to store files—it is the foundation for many AWS architectures.

In real-world environments, S3 is commonly used for:

- CI/CD artifact storage
- Static website hosting
- Log archival
- Data lakes
- Disaster recovery
- Cross-account backups
- Application configuration
- Lambda integrations
- Analytics pipelines

When designing an S3 solution, always evaluate:

1. **Security** – Encryption, IAM, Bucket Policies, Block Public Access.
2. **Performance** – Multipart Upload, CloudFront, Transfer Acceleration.
3. **Cost** – Lifecycle Rules, Storage Classes, Intelligent-Tiering.
4. **Availability** – Versioning, Replication, Multi-Region design.
5. **Operations** – Monitoring, Logging, Inventory, Automation.

A production-ready Amazon S3 environment is **secure, scalable, cost-optimized, monitored, and resilient**.

# End of Amazon S3 (Part 7)
