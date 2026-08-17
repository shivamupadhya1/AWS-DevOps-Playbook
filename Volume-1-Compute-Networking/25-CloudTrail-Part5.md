# AWS CloudTrail – Part 5

> AWS DevOps Playbook
>
> Volume 1 – Monitoring, Security & Governance
>
> Chapter 25
>
> AWS CloudTrail – Part 5 (Production Architecture, DevSecOps, Troubleshooting & Interview Master Guide)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Design an enterprise CloudTrail architecture
- Integrate CloudTrail with AWS security services
- Troubleshoot CloudTrail issues
- Understand real-world DevOps use cases
- Learn CloudTrail interview questions
- Master production best practices

---

# 116. Enterprise CloudTrail Architecture

A large enterprise may have:

- 100+ AWS Accounts
- Multiple AWS Regions
- Thousands of EC2 Instances
- Hundreds of Developers

Architecture

```
Developer

↓

AWS Console / CLI

↓

AWS APIs

↓

CloudTrail

↓

Organization Trail

↓

Central Logging Account

↓

Encrypted S3 Bucket

↓

CloudTrail Lake

↓

Athena

↓

CloudWatch

↓

Security Hub

↓

GuardDuty

↓

SOC Team
```

---

# 117. Why Centralized Logging?

Imagine every AWS account stores its own logs.

```
Account A

↓

S3 Bucket A

----------------

Account B

↓

S3 Bucket B

----------------

Account C

↓

S3 Bucket C
```

Searching becomes difficult.

Instead

```
All Accounts

↓

Central Logging Account

↓

Single S3 Bucket

↓

Easy Investigation
```

---

# 118. Multi-Account Logging

Production Setup

```
AWS Organizations

│

├── Dev Account

├── QA Account

├── UAT Account

├── Production Account

└── Shared Services

↓

Organization Trail

↓

Central Security Account
```

---

# 119. CloudTrail + CloudWatch

CloudTrail records

```
DeleteBucket
```

CloudWatch detects

```
Metric Filter

↓

Alarm

↓

SNS

↓

Email
```

Together they provide monitoring **and** auditing.

---

# 120. CloudTrail + EventBridge

CloudTrail events can trigger EventBridge rules.

Example

```
CreateUser

↓

CloudTrail

↓

EventBridge

↓

Lambda

↓

Send Slack/Teams Notification
```

Production teams use this for automation.

---

# 121. CloudTrail + Lambda

Example

```
Security Group Modified

↓

CloudTrail

↓

EventBridge

↓

Lambda

↓

Revert Change
```

This supports automated remediation.

---

# 122. CloudTrail + AWS Config

CloudTrail answers

```
WHO changed it?
```

AWS Config answers

```
WHAT changed?
```

Example

```
Security Group

↓

Port 22 Opened

↓

Config

↓

Resource Changed

↓

CloudTrail

↓

IAM User
```

---

# 123. CloudTrail + GuardDuty

GuardDuty continuously analyzes CloudTrail events.

Example

```
Compromised IAM Key

↓

Thousands of API Calls

↓

GuardDuty

↓

Security Finding
```

---

# 124. CloudTrail + Security Hub

```
CloudTrail

↓

GuardDuty

↓

AWS Config

↓

Inspector

↓

Security Hub

↓

Central Dashboard
```

Security Hub aggregates findings from multiple AWS security services.

---

# 125. CloudTrail + IAM

CloudTrail records:

- Login attempts
- Access key creation
- Policy attachment
- Role assumption
- Permission changes

This is critical for auditing privileged access.

---

# 126. DevOps Deployment Example

```
GitHub

↓

Jenkins

↓

Terraform

↓

AWS APIs

↓

CloudTrail

↓

Audit Logs
```

Every infrastructure deployment leaves an audit trail.

---

# 127. Kubernetes Example

Amazon EKS

```
kubectl apply

↓

AWS API

↓

CloudTrail
```

CloudTrail records AWS control plane operations such as cluster creation and node group changes. Kubernetes resource changes inside the cluster require Kubernetes audit logs, not CloudTrail.

---

# 128. Common Security Investigation

Incident

```
Production Server Deleted
```

Investigation

```
CloudTrail

↓

TerminateInstances

↓

IAM User

↓

Source IP

↓

MFA Used?

↓

Time

↓

Resolution
```

---

# 129. Troubleshooting Scenario 1

### Problem

CloudTrail logs are not appearing in S3.

Possible Causes

- Incorrect bucket policy
- Missing IAM permissions
- Wrong bucket name
- KMS permission issues
- Trail disabled

---

# 130. Troubleshooting Scenario 2

### Problem

CloudTrail cannot publish to CloudWatch Logs.

Possible Causes

- IAM Role missing
- Incorrect Log Group
- Region mismatch
- Permission denied

---

# 131. Troubleshooting Scenario 3

### Problem

Missing S3 Object Events

Reason

```
Data Events

Not Enabled
```

CloudTrail records bucket-level management events by default, but object-level events require Data Events to be configured.

---

# 132. Troubleshooting Scenario 4

### Problem

Athena query returns no results.

Check

- Correct S3 path
- Glue table
- Partition configuration
- Region
- Event time range

---

# 133. Production Best Practices

✔ Enable Organization Trail.

✔ Enable Multi-Region logging.

✔ Encrypt logs using KMS.

✔ Enable Log File Validation.

✔ Store logs in a dedicated logging account.

✔ Enable versioning on the logging bucket.

✔ Configure lifecycle policies.

✔ Monitor CloudTrail health.

✔ Enable Data Events only where needed.

✔ Periodically test audit processes.

---

# 134. Common Mistakes

❌ Using a Single Region Trail.

❌ Public S3 bucket.

❌ No KMS encryption.

❌ No lifecycle rules.

❌ No log validation.

❌ Never checking audit logs.

❌ Recording unnecessary Data Events.

❌ Allowing users to delete CloudTrail logs.

---

# 135. Production Checklist

Before going live, verify:

- [ ] Multi-Region Trail enabled
- [ ] Organization Trail configured
- [ ] Dedicated logging account
- [ ] S3 bucket encrypted
- [ ] KMS enabled
- [ ] Log File Validation enabled
- [ ] Versioning enabled
- [ ] Lifecycle policy configured
- [ ] CloudWatch integration configured
- [ ] GuardDuty enabled
- [ ] Security Hub enabled
- [ ] Access restricted using IAM

---

# 136. Real Interview Scenario

**Interviewer**

> Your production EC2 instance disappeared. How would you investigate?

### Answer

1. Check CloudTrail for the `TerminateInstances` event.
2. Identify the IAM user or assumed role.
3. Verify the source IP address.
4. Check whether MFA was used.
5. Review related CloudWatch alarms and logs.
6. Check deployment pipelines (Terraform/CloudFormation/Jenkins).
7. Review AWS Config for resource configuration changes.
8. Document the timeline and root cause.

---

# 137. Real Interview Scenario

**Interviewer**

> How do CloudTrail, CloudWatch and AWS Config work together?

### Answer

| Service | Purpose |
|----------|----------|
| CloudTrail | Records who performed AWS API actions |
| CloudWatch | Monitors resource health, logs and alarms |
| AWS Config | Tracks resource configuration and compliance |

Together they provide complete operational visibility.

---

# 138. 20 Rapid-Fire Interview Questions

### Q1. Does CloudTrail record Console actions?

**Yes.**

---

### Q2. Does it record CLI commands?

**Yes, through the underlying AWS API calls.**

---

### Q3. Does it record SDK calls?

**Yes.**

---

### Q4. Where are CloudTrail logs stored?

**Amazon S3 (when a Trail is configured).**

---

### Q5. What is Event History?

**Recent management events available without creating a Trail.**

---

### Q6. Difference between Event History and Trail?

**Event History is limited and intended for recent management events, while a Trail continuously records and delivers events for long-term storage and analysis.**

---

### Q7. What are Data Events?

**Object-level operations such as `GetObject`, `PutObject`, and Lambda invocations.**

---

### Q8. What are Management Events?

**Operations that manage AWS resources, such as creating or deleting EC2 instances.**

---

### Q9. What is CloudTrail Lake?

**A managed event data store with SQL-based querying capabilities.**

---

### Q10. What is Log File Validation?

**A feature that verifies CloudTrail log integrity and detects tampering.**

---

### Q11. Can CloudTrail detect attacks?

**It records activity. Services like GuardDuty analyze CloudTrail events to detect suspicious behavior.**

---

### Q12. What is an Organization Trail?

**A trail that records activity across AWS Organization accounts.**

---

### Q13. Does CloudTrail monitor CPU usage?

**No. CloudWatch monitors performance metrics such as CPU utilization.**

---

### Q14. Can CloudTrail trigger automation?

**Yes, by integrating with Amazon EventBridge and AWS Lambda.**

---

### Q15. Is CloudTrail regional?

**Trails can be Single-Region or Multi-Region.**

---

### Q16. Can CloudTrail record S3 object access?

**Yes, when Data Events are enabled.**

---

### Q17. Does Terraform activity appear in CloudTrail?

**Yes, because Terraform invokes AWS APIs.**

---

### Q18. Why encrypt CloudTrail logs?

**To protect sensitive audit records and satisfy compliance requirements.**

---

### Q19. Can CloudTrail logs be queried?

**Yes, using CloudTrail Lake or Amazon Athena.**

---

### Q20. Which AWS service should you check first after an unauthorized infrastructure change?

**AWS CloudTrail.**

---

# 139. Complete CloudTrail Mind Map

```
AWS CloudTrail

│

├── Event History

├── Trails

│     ├── Single Region

│     ├── Multi Region

│     └── Organization Trail

├── Management Events

├── Data Events

├── Insights Events

├── CloudTrail Lake

├── Event Selectors

├── Advanced Event Selectors

├── S3

├── KMS

├── Log Validation

├── Athena

├── CloudWatch Logs

├── EventBridge

├── Lambda

├── AWS Config

├── GuardDuty

├── Security Hub

├── Compliance

├── Incident Response

└── Root Cause Analysis
```

---

# 140. One-Page Revision

```
CloudTrail

↓

AWS API Calls

↓

Events

↓

Trail

↓

S3

↓

KMS

↓

CloudTrail Lake

↓

Athena

↓

CloudWatch

↓

EventBridge

↓

AWS Config

↓

GuardDuty

↓

Security Hub

↓

Compliance

↓

Security Investigation
```

---

# Think Like a Principal DevSecOps Engineer

CloudTrail is the **audit backbone** of every AWS environment.

A mature enterprise architecture should ensure:

1. **Every AWS account** is covered by an Organization Trail.
2. **Every API call** is securely recorded.
3. **Audit logs** are encrypted, immutable, and centrally stored.
4. **Critical events** trigger automated alerts and remediation.
5. **Security teams** can investigate incidents within minutes using CloudTrail, Athena, and CloudTrail Lake.
6. **Compliance teams** can prove who changed what, when, and from where.
7. **Developers, DevOps, and Security** all rely on the same trusted audit data.

CloudTrail doesn't prevent mistakes—it ensures that every action is **traceable, verifiable, and accountable**, making it one of the most important governance services in AWS.

# End of AWS CloudTrail (5 Parts)