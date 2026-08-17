# Amazon S3

> AWS DevOps Playbook
>
> Volume 1 – Storage
>
> Chapter 22
>
> Amazon S3 – Part 3 (Security, Versioning & Access Control)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand how S3 security works
- Learn IAM Policies
- Learn Bucket Policies
- Understand ACLs
- Learn Block Public Access
- Understand Object Ownership
- Learn Versioning
- Understand MFA Delete
- Learn Cross Account Access
- Design secure production architectures
- Answer interview questions

---

# 54. Why S3 Security is Important

One of the biggest causes of cloud data breaches is:

**Publicly accessible S3 buckets**

Example:

```
Developer

↓

Creates Bucket

↓

Allows Public Access

↓

Sensitive Files

↓

Internet
```

Thousands of companies have accidentally exposed:

- Customer data
- Employee records
- Source code
- Database backups
- API Keys

AWS therefore provides multiple security layers.

---

# 55. S3 Security Layers

```
                    Amazon S3

                        │

        ┌───────────────┼─────────────────┐

        │               │                 │

      IAM          Bucket Policy       ACL

        │

        ▼

Block Public Access

        │

        ▼

Object Ownership

        │

        ▼

Encryption

        │

        ▼

Versioning
```

In production, multiple security mechanisms work together.

---

# 56. IAM Policy

IAM controls **who can perform which actions on S3 resources**.

Example:

```
Developer

↓

IAM Policy

↓

Allow

↓

Read Bucket
```

Example Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect":"Allow",
      "Action":[
        "s3:GetObject"
      ],
      "Resource":"arn:aws:s3:::company-data/*"
    }
  ]
}
```

This allows downloading objects.

---

# 57. Bucket Policy

A Bucket Policy is a **resource-based policy** attached directly to the bucket.

Instead of attaching permissions to a user,

you attach permissions to the bucket.

Example

```
Bucket

↓

Policy

↓

Allow

↓

Account B
```

---

Example

```json
{
 "Version":"2012-10-17",
 "Statement":[
   {
     "Effect":"Allow",
     "Principal":"*",
     "Action":"s3:GetObject",
     "Resource":"arn:aws:s3:::mybucket/*"
   }
 ]
}
```

⚠️ This example makes objects publicly readable and should **not** be used for sensitive data.

---

# 58. IAM Policy vs Bucket Policy

| IAM Policy | Bucket Policy |
|------------|---------------|
| Attached to User/Role | Attached to Bucket |
| Identity-based | Resource-based |
| Controls what a principal can do | Controls who can access the bucket |
| Used within an account | Commonly used for cross-account access or public access (when appropriate) |

---

# 59. Access Control List (ACL)

ACL is the **older access control mechanism**.

It allows permissions on:

- Bucket
- Individual Objects

Example

```
Bucket

↓

Object

↓

ACL

↓

Read

↓

Account B
```

---

# 60. Why AWS Recommends Disabling ACLs

Modern AWS best practice:

```
IAM

+

Bucket Policy

+

Object Ownership

=

No ACL Required
```

Reasons:

- Simpler permission model
- Easier troubleshooting
- Better security
- Consistent ownership

Most new AWS environments disable ACLs.

---

# 61. Block Public Access

One of the most important S3 security features.

```
Internet

↓

Bucket

↓

Blocked
```

Block Public Access prevents accidental public exposure.

AWS recommends enabling it for almost all buckets unless public access is intentionally required.

---

# 62. Four Block Public Access Settings

AWS provides four controls:

```
✔ Block Public ACLs

✔ Ignore Public ACLs

✔ Block Public Bucket Policies

✔ Restrict Public Bucket Policies
```

These can be applied:

- Account level
- Bucket level

---

# 63. Example

Suppose someone uploads:

```
salary.xlsx
```

and mistakenly grants:

```
Everyone

Read Access
```

If Block Public Access is enabled:

```
Public Access

↓

Denied
```

The object remains private.

---

# 64. Object Ownership

Historically:

Uploader owned the object.

Example:

```
Account A

↓

Uploads

↓

Bucket of Account B

↓

Object Owner = Account A
```

This created permission issues.

---

# 65. Bucket Owner Enforced

Modern AWS recommends:

```
Bucket Owner Enforced
```

Benefits:

- Bucket owner owns every object.
- ACLs are disabled.
- Easier permission management.
- Simplified cross-account uploads.

This is the recommended default for new buckets.

---

# 66. Versioning

Versioning protects against:

- Accidental deletion
- Accidental overwrite
- Ransomware
- Human mistakes

Example

```
report.pdf

↓

Version 1

↓

Version 2

↓

Version 3
```

Instead of replacing objects, S3 stores new versions.

---

# 67. Versioning Example

```
Upload

resume.pdf

↓

Version 1

----------------

Upload Again

↓

Version 2

----------------

Upload Again

↓

Version 3
```

All versions are retained until deleted according to your retention policy.

---

# 68. Benefits of Versioning

✔ Recover deleted files

✔ Recover overwritten files

✔ Protect against accidental changes

✔ Improve disaster recovery

✔ Support replication features

---

# 69. Delete Marker

When versioning is enabled and you delete an object:

```
Delete

↓

Delete Marker Created

↓

Old Versions Still Exist
```

The latest version becomes a delete marker.

Older versions remain unless permanently removed.

---

# 70. Recovering Deleted Objects

Example

```
Version 1

↓

Version 2

↓

Delete Marker
```

Recovery:

Delete the delete marker.

The latest object version becomes visible again.

---

# 71. MFA Delete

MFA Delete provides extra protection.

Deleting versions requires:

```
Password

+

MFA Token
```

Useful for:

- Financial systems
- Compliance
- Critical backups

Note: MFA Delete has specific administrative requirements and is typically managed using the AWS CLI or API.

---

# 72. Cross Account Access

Example:

```
Account A

↓

Bucket

↓

Bucket Policy

↓

Allow

↓

Account B
```

Common use cases:

- Shared log bucket
- Central backup bucket
- Organization-wide data lake
- CI/CD artifact sharing

---

# 73. Production Example

Organization

```
Security Account

↓

Central Log Bucket

↓

Bucket Policy

↓

Allow

↓

Dev Account

↓

Prod Account

↓

QA Account
```

All accounts write logs into one central bucket.

---

# 74. Best Practices

✔ Enable Block Public Access by default.

✔ Enable Versioning.

✔ Use Bucket Owner Enforced.

✔ Disable ACLs where possible.

✔ Grant least-privilege IAM permissions.

✔ Use Bucket Policies only when needed.

✔ Review bucket access regularly.

✔ Enable server-side encryption (covered in Part 4).

---

# 75. Common Mistakes

❌ Public buckets containing sensitive data.

❌ Using ACLs unnecessarily.

❌ No Versioning.

❌ Granting `"Principal": "*"` without restrictions.

❌ Disabling Block Public Access without understanding the impact.

❌ Giving `s3:*` permissions to every user.

---

# 76. Production Scenario 1

### Problem

Developer accidentally deletes:

```
customer-data.csv
```

Solution:

Recover the previous version because Versioning is enabled.

---

# 77. Production Scenario 2

### Problem

Security audit finds:

```
Public Bucket
```

Root Cause:

Block Public Access disabled.

Solution:

- Enable Block Public Access.
- Remove public Bucket Policy.
- Review IAM permissions.

---

# 78. Production Scenario 3

### Problem

Another AWS account needs to upload logs.

Solution:

Use a Bucket Policy granting the required permissions to the external account, and configure Object Ownership appropriately.

---

# 79. Production Scenario 4

### Problem

Multiple accounts upload objects.

Ownership conflicts occur.

Solution:

Use:

```
Bucket Owner Enforced
```

This ensures the bucket owner owns all uploaded objects.

---

# 80. Interview Questions

## Question 17

What is the difference between an IAM Policy and a Bucket Policy?

### Answer

IAM Policies are attached to users, groups, or roles and define what those identities can do. Bucket Policies are attached directly to the bucket and define who can access the bucket and under what conditions.

---

## Question 18

Why should Block Public Access be enabled?

### Answer

It helps prevent accidental exposure of buckets and objects to the public Internet.

---

## Question 19

Why is Versioning important?

### Answer

Versioning protects against accidental deletion, overwrites, and helps recover previous versions of objects.

---

## Question 20

What happens when you delete an object from a version-enabled bucket?

### Answer

S3 creates a delete marker. Previous object versions remain stored until they are explicitly removed.

---

## Question 21

Should ACLs be used in new AWS environments?

### Answer

Generally no. AWS recommends using IAM Policies, Bucket Policies, and Bucket Owner Enforced instead of ACLs.

---

## Question 22

What is Bucket Owner Enforced?

### Answer

It is an Object Ownership setting that disables ACLs and ensures the bucket owner automatically owns all objects in the bucket.

---

## Question 23

Can two AWS accounts access the same bucket?

### Answer

Yes. Cross-account access can be configured using Bucket Policies, IAM roles, or other AWS access mechanisms.

---

# 81. Hands-on Labs

## Lab 11

Enable Versioning on a bucket.

Upload multiple versions of the same file.

---

## Lab 12

Delete an object and restore it using Versioning.

---

## Lab 13

Create an IAM Policy that allows only:

- List Bucket
- Get Object

---

## Lab 14

Create a Bucket Policy allowing another AWS account to upload objects.

---

## Lab 15

Enable Bucket Owner Enforced and verify that ACLs are disabled.

---

# 82. One-Page Revision

```
IAM Policy

↓

Bucket Policy

↓

Block Public Access

↓

Object Ownership

↓

Versioning

↓

MFA Delete

↓

Cross Account Access
```

Remember:

- IAM → Identity permissions
- Bucket Policy → Resource permissions
- ACL → Legacy (avoid if possible)
- Block Public Access → Keep enabled
- Versioning → Recover deleted/overwritten objects
- Delete Marker ≠ Permanent deletion
- Bucket Owner Enforced → Recommended default

---

# Think Like a Production Engineer

The majority of S3 security incidents are caused by **misconfiguration**, not by flaws in S3 itself.

When designing production environments:

1. Keep buckets private by default.
2. Enable Block Public Access.
3. Use IAM Roles and Bucket Policies with least privilege.
4. Enable Versioning for important buckets.
5. Disable ACLs using Bucket Owner Enforced.
6. Regularly audit bucket permissions using AWS IAM Access Analyzer and AWS Config.
7. Encrypt sensitive data and monitor access logs.

A secure S3 bucket should be **private, encrypted, versioned, monitored, and accessible only to authorized identities**.

# End of Part 3
