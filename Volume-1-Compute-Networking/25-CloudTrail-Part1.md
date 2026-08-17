# AWS CloudTrail – Part 1

> AWS DevOps Playbook
>
> Volume 1 – Monitoring, Security & Governance
>
> Chapter 25
>
> AWS CloudTrail – Part 1 (Introduction, Events, Trails & Audit Fundamentals)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand AWS CloudTrail
- Differentiate CloudTrail and CloudWatch
- Understand AWS API Calls
- Learn Event History
- Understand Trails
- Learn Management Events
- Learn Data Events
- Learn Insights Events
- Understand Event JSON
- Learn Production Use Cases
- Answer Interview Questions

---

# 1. What is AWS CloudTrail?

AWS CloudTrail is an **auditing and governance service** that records every API activity performed inside your AWS account.

It answers questions like:

- Who created an EC2 instance?
- Who deleted an S3 bucket?
- Who modified a Security Group?
- When was an IAM user created?
- Which IP address made the request?

Think of CloudTrail as the **audit log** of your AWS account.

---

# 2. Why Do We Need CloudTrail?

Imagine your production server suddenly disappears.

Without CloudTrail

```
EC2 Deleted

↓

Nobody Knows Who Deleted It
```

With CloudTrail

```
EC2 Deleted

↓

User: Shivam

↓

Time: 11:32 AM

↓

Source IP

↓

DeleteInstances API
```

Everything is recorded.

---

# 3. Real Production Scenario

Suppose a production database is deleted.

Management asks:

- Who deleted it?
- From which IP?
- At what time?
- Using which IAM user?
- From AWS Console or CLI?

CloudTrail answers all of these.

---

# 4. CloudTrail vs CloudWatch

Many beginners confuse them.

| CloudTrail | CloudWatch |
|------------|------------|
| Auditing | Monitoring |
| API Calls | Metrics |
| User Activity | CPU |
| Security | Memory |
| Governance | Logs |
| Compliance | Performance |

Remember:

```
CloudTrail

↓

WHO did it?
```

```
CloudWatch

↓

WHAT happened?
```

---

# 5. What Does CloudTrail Record?

CloudTrail records API calls from:

- AWS Console
- AWS CLI
- AWS SDK
- CloudFormation
- Terraform
- AWS Services

Example

```
Terraform Apply

↓

Create EC2

↓

CloudTrail Records API
```

---

# 6. CloudTrail Architecture

```
User

↓

AWS Console

↓

Create EC2

↓

EC2 API

↓

CloudTrail

↓

Trail

↓

S3

↓

Athena

↓

CloudWatch
```

---

# 7. CloudTrail Components

```
CloudTrail

│

├── Event History

├── Trails

├── Management Events

├── Data Events

├── Insights Events

├── CloudTrail Lake

├── Event Selectors

└── Log File Validation
```

---

# 8. What is an Event?

Every AWS API call is an Event.

Example

```
CreateBucket

↓

One Event
```

Another

```
RunInstances

↓

One Event
```

Another

```
TerminateInstances

↓

One Event
```

---

# 9. Event Lifecycle

```
User

↓

API Call

↓

CloudTrail

↓

Event

↓

Storage

↓

Search
```

---

# 10. Event History

CloudTrail automatically stores recent management events in **Event History**.

Features:

- Searchable
- No setup required
- Recent activity
- Available directly in the AWS Console

Useful during quick investigations.

---

# 11. What is a Trail?

A Trail continuously delivers CloudTrail events to a destination (typically an S3 bucket).

Architecture

```
CloudTrail

↓

Trail

↓

S3 Bucket

↓

Permanent Storage
```

Without a Trail, long-term storage and centralized logging are not configured.

---

# 12. Why Create a Trail?

Benefits:

✔ Long-term storage

✔ Compliance

✔ Audit history

✔ Investigation

✔ Integration with Athena

✔ Centralized logging

---

# 13. Types of Events

CloudTrail supports:

```
Management Events

↓

Data Events

↓

Insights Events
```

---

# 14. Management Events

Management Events record changes to AWS resources.

Examples:

- Create EC2
- Stop EC2
- Delete IAM User
- Modify Security Group
- Create VPC
- Update Route Table
- Attach IAM Policy

These are enabled by default in Event History.

---

# 15. Real Example

```
AWS Console

↓

Launch EC2

↓

RunInstances API

↓

CloudTrail Event
```

---

# 16. Data Events

Data Events record operations performed **inside** resources.

Examples

```
S3

↓

GetObject

PutObject

DeleteObject
```

Another

```
Lambda

↓

Invoke Function
```

These are **not enabled by default** because they can generate a very large number of events.

---

# 17. Example

```
Upload File

↓

PutObject

↓

CloudTrail Event
```

---

# 18. Insights Events

CloudTrail Insights detects unusual API activity.

Example

Normal

```
20 API Calls
```

Suddenly

```
5000 API Calls
```

CloudTrail Insights detects the anomaly and generates an Insight event.

---

# 19. Why Insights Matter

Suppose an IAM access key is compromised.

Normally

```
100 API Calls
```

Suddenly

```
15,000 API Calls
```

CloudTrail Insights can identify this abnormal behavior.

---

# 20. Event Structure

Every CloudTrail event contains information such as:

- Event Time
- Event Name
- Event Source
- User Identity
- Region
- Source IP
- Request Parameters
- Response Elements

---

# 21. Sample Event

```
Event Name

RunInstances

User

admin-user

Region

ap-south-1

Source IP

49.x.x.x

Time

11:35 AM
```

---

# 22. CloudTrail Event JSON

Every event is stored in JSON format.

Example

```json
{
  "eventTime":"2026-08-17T10:30:00Z",
  "eventName":"RunInstances",
  "awsRegion":"ap-south-1",
  "sourceIPAddress":"49.x.x.x",
  "userIdentity":{
      "type":"IAMUser",
      "userName":"admin-user"
  }
}
```

Understanding this structure is important for troubleshooting and audits.

---

# 23. Regions

CloudTrail can operate as:

```
Single Region Trail
```

or

```
Multi-Region Trail
```

Multi-Region Trails are recommended for production.

---

# 24. Global Services

Some AWS services are global.

Examples:

- IAM
- AWS Organizations
- Route 53

CloudTrail records events for these as well.

---

# 25. Real DevOps Example

Suppose someone opens:

```
Security Group

↓

Allows

0.0.0.0/0

↓

Port 22
```

CloudTrail records:

- Who modified it
- When it was modified
- IP Address
- IAM User
- Request Details

---

# 26. Best Practices

✔ Enable Multi-Region Trails.

✔ Store logs in S3.

✔ Encrypt logs.

✔ Enable Log File Validation.

✔ Restrict S3 bucket access.

✔ Use IAM Roles.

✔ Monitor unusual API activity.

---

# 27. Common Mistakes

❌ No Trail configured.

❌ Single Region Trail only.

❌ Public S3 bucket for logs.

❌ No encryption.

❌ No log validation.

❌ Never reviewing audit logs.

---

# 28. Interview Questions

## Question 1

What is AWS CloudTrail?

### Answer

AWS CloudTrail is an auditing service that records AWS API calls made by users, applications, and AWS services for governance, compliance, and security investigations.

---

## Question 2

What is the difference between CloudTrail and CloudWatch?

### Answer

CloudTrail records **who performed an action** in AWS, while CloudWatch monitors the **health and performance** of AWS resources through metrics, logs, and alarms.

---

## Question 3

What is a Trail?

### Answer

A Trail continuously delivers CloudTrail events to a configured destination such as an Amazon S3 bucket for long-term storage and analysis.

---

## Question 4

What are Management Events?

### Answer

Management Events record operations that manage AWS resources, such as creating EC2 instances, modifying IAM policies, or updating Security Groups.

---

## Question 5

What are Data Events?

### Answer

Data Events record operations performed within resources, such as S3 object-level operations (`GetObject`, `PutObject`) or Lambda function invocations.

---

## Question 6

What are CloudTrail Insights?

### Answer

CloudTrail Insights identify unusual API activity by detecting anomalies compared to normal usage patterns.

---

# 29. Hands-on Labs

## Lab 1

Open **CloudTrail → Event History** and identify:

- EC2 launches
- IAM logins
- S3 bucket creation

---

## Lab 2

Create a Multi-Region Trail.

---

## Lab 3

Configure an S3 bucket to store CloudTrail logs.

---

## Lab 4

Generate an EC2 launch event and verify it appears in CloudTrail.

---

# 30. One-Page Revision

```
AWS CloudTrail

↓

API Calls

↓

Events

↓

Management Events

↓

Data Events

↓

Insights Events

↓

Trail

↓

S3

↓

Audit

↓

Compliance

↓

Security Investigation
```

Remember:

- CloudTrail answers **who did what, when, and from where**.
- Every AWS API call becomes an event.
- Trails store events for long-term auditing.
- Management Events track resource changes.
- Data Events track activity inside resources.
- Insights detect abnormal API behavior.

---

# Think Like a Senior DevOps Engineer

CloudTrail is the foundation of AWS auditing and security.

In a production environment:

1. Enable a **Multi-Region Trail**.
2. Store logs securely in an encrypted S3 bucket.
3. Protect logs from modification.
4. Enable Log File Validation.
5. Regularly review API activity for security incidents.
6. Integrate CloudTrail with monitoring and analytics tools.

Monitoring tells you **what happened**. CloudTrail tells you **who caused it**. Together, they provide complete operational visibility across your AWS environment.

# End of Part 1