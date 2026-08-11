# IAM – Part 2 (Policy Evaluation & Advanced Permissions)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 18
>
> IAM – Part 2

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand IAM Policy Evaluation Logic
- Explain Explicit vs Implicit Deny
- Understand Identity-based Policies
- Understand Resource-based Policies
- Understand Permission Boundaries
- Understand Session Policies
- Use Policy Conditions
- Troubleshoot AccessDenied errors
- Answer Senior DevOps Interview Questions

---

# 1. How AWS Decides Whether to Allow a Request

This is the most important IAM topic.

Whenever an AWS API request is made:

```
User / Role

↓

AWS Authentication

↓

Collect All Applicable Policies

↓

Evaluate Policies

↓

Allow OR Deny
```

AWS does **not** stop after checking one policy.

It evaluates every applicable policy.

---

# 2. IAM Policy Evaluation Flow

```
Authentication

↓

Identity Policy

↓

Resource Policy

↓

Permission Boundary

↓

Session Policy

↓

Organizations SCP

↓

Explicit Deny?

↓

ALLOW / DENY
```

Remember:

AWS combines multiple policy types before making a decision.

---

# 3. IAM Policy Types

AWS evaluates several kinds of policies.

### Identity-based Policies

Attached to:

- User
- Group
- Role

Example:

```
Developer Role

↓

AmazonS3ReadOnly
```

---

### Resource-based Policies

Attached directly to a resource.

Examples:

- S3 Bucket Policy
- KMS Key Policy
- SQS Queue Policy
- SNS Topic Policy

Example:

```
S3 Bucket

↓

Bucket Policy

↓

Allow Account B
```

---

### Permission Boundaries

Attached to:

```
User

Role
```

Acts as the **maximum permissions** an identity can ever receive.

Think of it as a permission ceiling.

---

### Session Policies

Applied only to temporary credentials obtained from STS.

They can further restrict the permissions available during that session.

---

### Service Control Policies (SCP)

Applied through AWS Organizations.

They define the **maximum permissions available within an account or Organizational Unit (OU)**.

Even if IAM allows an action, an SCP can block it.

---

# 4. Implicit Deny

Every request starts as:

```
DENY
```

This is called **Implicit Deny**.

Nothing is allowed until an Allow statement matches.

Example:

```
User

↓

No Policies

↓

Launch EC2
```

Result:

```
Denied
```

---

# 5. Explicit Deny

Explicit Deny always overrides Allow.

Example

Policy A

```
Allow

s3:*
```

Policy B

```
Deny

s3:DeleteObject
```

Result

```
DeleteObject

↓

Denied
```

Even though another policy allows all S3 actions.

---

# 6. IAM Evaluation Rules

Remember these four rules:

1.

Default = Implicit Deny

2.

At least one Allow must match.

3.

Any Explicit Deny overrides all Allows.

4.

All applicable policies are evaluated together.

---

# 7. Identity Policy Example

```json
{
  "Effect":"Allow",
  "Action":"ec2:StartInstances",
  "Resource":"*"
}
```

This allows starting EC2 instances.

---

# 8. Resource Policy Example (S3 Bucket)

```json
{
 "Version":"2012-10-17",
 "Statement":[
   {
     "Effect":"Allow",
     "Principal":{
       "AWS":"arn:aws:iam::111111111111:role/AppRole"
     },
     "Action":"s3:GetObject",
     "Resource":"arn:aws:s3:::company-data/*"
   }
 ]
}
```

The bucket itself grants access to a specific role.

---

# 9. Identity Policy + Resource Policy

Suppose:

Identity Policy

```
Allow GetObject
```

Bucket Policy

```
Allow Role
```

Result

```
Access Allowed
```

Now:

Bucket Policy

```
Explicit Deny
```

Result

```
Denied
```

Resource policy Deny overrides.

---

# 10. Permission Boundary

Suppose:

Developer Role

Policy

```
Allow EC2
Allow S3
Allow RDS
```

Permission Boundary

```
Allow EC2 Only
```

Effective permissions:

```
EC2

Allowed

S3

Denied

RDS

Denied
```

Permission boundaries limit the maximum permissions.

---

# 11. Why Permission Boundaries Exist

Imagine developers can create IAM Roles.

Without boundaries:

Developer creates

```
AdministratorAccess
```

Now they become admin.

Permission Boundary prevents privilege escalation.

---

# 12. Session Policies

Used with:

- STS AssumeRole
- Federation
- IAM Identity Center

Example

Role

```
Administrator
```

Session Policy

```
ReadOnly S3
```

During that session:

Only ReadOnly S3 permissions are available.

---

# 13. IAM Conditions

Conditions make policies dynamic.

Example:

```
Allow

↓

Only

↓

From Office IP
```

---

Example

```json
"Condition":{
   "IpAddress":{
      "aws:SourceIp":"203.0.113.0/24"
   }
}
```

---

# 14. Common IAM Condition Keys

| Condition | Purpose |
|-----------|----------|
| aws:SourceIp | Restrict by IP |
| aws:CurrentTime | Time-based access |
| aws:MultiFactorAuthPresent | Require MFA |
| aws:PrincipalTag | Tag-based access |
| aws:ResourceTag | Resource tag control |
| aws:SecureTransport | Force HTTPS |

---

# 15. MFA Example

Allow deleting S3 objects only if MFA is present.

```json
"Condition":{
  "Bool":{
     "aws:MultiFactorAuthPresent":"true"
  }
}
```

---

# 16. Tag-Based Access Control (ABAC)

Instead of granting permissions by user names, use tags.

Example:

Developer Tag

```
Department=Dev
```

EC2 Tag

```
Department=Dev
```

Policy:

Allow access only when the tags match.

Useful in large organizations.

---

# 17. Production Access Flow

```
Developer

↓

IAM Role

↓

Permission Boundary

↓

Session Policy

↓

SCP

↓

S3 Bucket Policy

↓

AWS Decision
```

---

# 18. Common AccessDenied Reasons

- Missing Allow statement
- Explicit Deny
- Wrong Resource ARN
- Missing IAM Role
- Bucket Policy Deny
- KMS Key Policy
- Permission Boundary restriction
- SCP restriction
- Session Policy restriction
- Missing trust relationship (for AssumeRole)

---

# 19. Best Practices

- Follow Least Privilege.
- Use Customer Managed Policies.
- Prefer Roles over Users.
- Use Permission Boundaries for delegated administration.
- Require MFA for sensitive actions.
- Use Conditions whenever possible.
- Review policies regularly.

---

# 20. Common Mistakes

❌ Using `"Action":"*"`

❌ Using `"Resource":"*"` unnecessarily

❌ Ignoring Explicit Deny

❌ Hardcoding ARNs incorrectly

❌ Forgetting Bucket Policies

❌ Ignoring SCP restrictions

---

# 21. Production Scenarios

## Scenario 1

### Problem

EC2 cannot read an S3 bucket.

Checklist:

- IAM Role attached?
- Identity Policy allows `s3:GetObject`?
- Bucket Policy allows access?
- KMS permissions?
- SCP restrictions?

---

## Scenario 2

### Problem

Developer has `AdministratorAccess` but still gets AccessDenied.

Possible causes:

- Permission Boundary
- Organizations SCP
- Resource Policy Explicit Deny

---

## Scenario 3

### Problem

Cross-account AssumeRole fails.

Check:

- Trust Policy
- IAM Permission to call `sts:AssumeRole`
- Session Policy
- SCP

---

## Scenario 4

### Problem

Lambda cannot decrypt Secrets Manager secret.

Check:

- Execution Role
- KMS Key Policy
- Secret Resource Policy

---

# 22. Troubleshooting Flow

```
Authentication

↓

Identity Policy

↓

Resource Policy

↓

Permission Boundary

↓

Session Policy

↓

SCP

↓

Explicit Deny?

↓

Allow / Deny
```

---

# 23. Interview Questions

## Question 1

What is Implicit Deny?

### Answer

Every AWS request is denied by default until at least one applicable Allow policy grants access.

---

## Question 2

What is Explicit Deny?

### Answer

An Explicit Deny always overrides any Allow from any applicable policy.

---

## Question 3

What is a Permission Boundary?

### Answer

A Permission Boundary defines the maximum permissions an IAM User or Role can have, even if broader permissions are attached.

---

## Question 4

Difference between Identity Policy and Resource Policy?

### Answer

Identity Policies are attached to IAM identities (users, groups, roles). Resource Policies are attached directly to AWS resources like S3 buckets, KMS keys, or SQS queues.

---

## Question 5

Can an S3 Bucket Policy grant access to another AWS account?

### Answer

Yes.

S3 Bucket Policies are commonly used for cross-account access.

---

## Question 6

What happens if one policy allows and another explicitly denies?

### Answer

Explicit Deny wins.

---

## Question 7

What is ABAC?

### Answer

Attribute-Based Access Control (ABAC) uses tags on users and resources to make authorization decisions instead of relying only on identity names.

---

## Question 8

Why use IAM Conditions?

### Answer

Conditions provide additional restrictions such as IP address, time of day, MFA status, tags, or secure transport requirements.

---

# 24. Amazon Follow-up Questions

### Question

Can a Resource Policy override an Explicit Deny in an Identity Policy?

### Answer

No.

An Explicit Deny in any applicable policy overrides all Allows.

---

### Question

Can a Permission Boundary grant permissions?

### Answer

No.

It only limits the maximum permissions that can be granted by identity-based policies.

---

### Question

Can an SCP grant permissions?

### Answer

No.

An SCP sets the maximum allowed permissions for accounts in an AWS Organization. IAM policies must still explicitly allow the action.

---

### Question

Why is my IAM policy correct but I still receive AccessDenied?

### Answer

Because another layer—such as an SCP, Permission Boundary, Session Policy, Resource Policy, KMS Key Policy, or an Explicit Deny—may be restricting the request.

---

# 25. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Create a Customer Managed Policy allowing only:

- `ec2:StartInstances`
- `ec2:StopInstances`

Attach it to a test role and verify behavior.

---

## Lab 2

Create an S3 Bucket Policy granting read access to another IAM Role.

Verify cross-role access.

---

## Lab 3

Apply a Permission Boundary to a developer role and confirm it cannot exceed the boundary even if broader permissions are attached.

---

## Lab 4

Create an IAM policy requiring MFA for deleting S3 objects using the `aws:MultiFactorAuthPresent` condition.

---

## Lab 5

Use the IAM Policy Simulator to test different Allow and Deny combinations before deploying policies.

---

# 26. One-Page Revision

```
Request
   │
Authenticate
   │
Identity Policy
   │
Resource Policy
   │
Permission Boundary
   │
Session Policy
   │
SCP
   │
Explicit Deny?
   │
ALLOW / DENY
```

Remember:

- Default = Implicit Deny
- At least one Allow is required
- Explicit Deny always wins
- Identity and Resource Policies work together
- Permission Boundaries restrict maximum permissions
- SCPs restrict accounts in AWS Organizations
- Conditions provide context-aware access control

---

# Think Like a Production Engineer

When troubleshooting **AccessDenied**, don't stop after checking the IAM policy.

Follow this order:

1. Verify the caller (User or Role).
2. Check Identity-based Policies.
3. Check Resource-based Policies.
4. Check Permission Boundaries.
5. Check Session Policies.
6. Check AWS Organizations SCPs.
7. Check for Explicit Deny.
8. Verify the correct resource ARN and required KMS permissions if encryption is involved.

This systematic approach is how experienced AWS engineers isolate IAM authorization issues in production.

# End of Part 2
