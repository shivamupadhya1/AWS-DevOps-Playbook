# AWS Identity and Access Management (IAM)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 17
>
> IAM – Part 1 (Fundamentals)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand IAM architecture
- Understand Authentication vs Authorization
- Understand Users
- Understand Groups
- Understand Roles
- Understand Policies
- Apply Least Privilege
- Design IAM for Production
- Answer Interview Questions

---

# 1. What is IAM?

AWS Identity and Access Management (IAM) is the service responsible for **authentication and authorization** in AWS.

It answers two questions:

```
Who are you?

↓

Authentication

-----------------------

What are you allowed to do?

↓

Authorization
```

Every request made to AWS is evaluated through IAM.

---

# 2. Why Do We Need IAM?

Imagine your AWS account contains:

- EC2
- S3
- RDS
- Lambda
- Secrets Manager

Should every engineer have full access?

Obviously not.

Example

```
Developer

↓

EC2
```

Allowed.

```
Developer

↓

Delete Production RDS
```

Denied.

IAM ensures users get only the permissions they need.

---

# 3. IAM Architecture

```
User

↓

IAM

↓

Authentication

↓

Authorization

↓

AWS Service
```

---

# 4. Authentication vs Authorization

Authentication

```
Who are you?
```

Examples:

- Username & Password
- Access Key
- MFA
- Temporary Credentials

Authorization

```
What can you do?
```

Examples:

- Launch EC2
- Read S3
- Delete RDS
- Create IAM User

---

# 5. IAM Components

Core IAM components:

- Users
- Groups
- Roles
- Policies

Everything in IAM revolves around these four concepts.

---

# 6. IAM User

An IAM User represents a single identity.

Examples:

```
shivam

john

alice
```

Each IAM User can have:

- Password
- Access Keys
- MFA Device
- Policies
- Group Membership

---

# 7. IAM Group

A Group is a collection of IAM Users.

Example

```
Developers

├── John

├── Alice

└── Bob
```

Assign permissions to the group instead of each user individually.

---

# 8. Benefits of Groups

Without Groups

```
100 Users

↓

100 Policies
```

With Groups

```
100 Users

↓

Developers Group

↓

1 Policy
```

Much easier to manage.

---

# 9. IAM Role

A Role is **not tied to a person**.

It is assumed temporarily by:

- EC2
- Lambda
- ECS
- EKS
- AWS Services
- IAM Users
- Other AWS Accounts

Roles provide **temporary credentials**.

---

# 10. Example of a Role

Application running on EC2 needs to access S3.

Wrong approach

```
Access Key

↓

Store in Application
```

Correct approach

```
EC2

↓

IAM Role

↓

Temporary Credentials

↓

S3
```

Never hardcode AWS credentials in applications.

---

# 11. IAM Policy

Policies define permissions.

They are written in JSON.

Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "*"
    }
  ]
}
```

Policies answer:

```
Who

↓

Can Perform

↓

Which Action

↓

On Which Resource
```

---

# 12. Policy Components

Every policy contains:

| Field | Purpose |
|--------|----------|
| Effect | Allow or Deny |
| Action | API Operations |
| Resource | Target Resource |
| Condition | Optional Restrictions |

---

# 13. Managed Policies

AWS provides predefined policies.

Examples:

```
AmazonS3ReadOnlyAccess

AmazonEC2FullAccess

AdministratorAccess
```

Advantages

- Easy to use
- Maintained by AWS
- Updated automatically

---

# 14. Customer Managed Policies

Created by your organization.

Advantages:

- Custom permissions
- Better security
- Least Privilege
- Reusable

Production environments commonly use customer-managed policies.

---

# 15. Inline Policies

Inline Policies are attached directly to a single user, group, or role.

Characteristics:

- One-to-one relationship
- Not reusable
- Deleted with the identity

Generally avoid them unless the permission is unique.

---

# 16. Principle of Least Privilege

One of the most important AWS security principles.

Grant:

```
Only

↓

The Minimum Permissions

↓

Required

↓

To Perform The Job
```

Example

Developer

Needs

```
Read S3
```

Do NOT grant

```
AdministratorAccess
```

---

# 17. Root User

Every AWS account has one Root User.

Capabilities:

- Full access
- Cannot be restricted by IAM policies

Best Practices:

- Enable MFA
- Do not create Access Keys
- Do not use for daily work
- Store credentials securely

---

# 18. IAM Credentials

IAM Users can authenticate using:

- Console Password
- Access Key ID
- Secret Access Key
- MFA Device

Applications should generally use IAM Roles instead of long-term access keys.

---

# 19. IAM in Production

Example Architecture

```
Developer

↓

IAM User

↓

Developer Group

↓

Policy

↓

EC2
```

Application

```
EC2

↓

IAM Role

↓

S3
```

Database Backup

```
Lambda

↓

IAM Role

↓

S3
```

---

# 20. Best Practices

- Enable MFA for all users.
- Avoid using the Root User.
- Use IAM Roles instead of Access Keys.
- Follow Least Privilege.
- Rotate credentials where applicable.
- Use Groups for permission management.
- Prefer Customer Managed Policies for production.

---

# 21. Common Mistakes

❌ Using the Root User daily.

❌ Sharing IAM User credentials.

❌ Hardcoding Access Keys.

❌ Giving AdministratorAccess to everyone.

❌ Creating one IAM User for an application instead of using a Role.

❌ Not enabling MFA.

---

# 22. Production Scenarios

## Scenario 1

### Problem

Application cannot access S3.

Check:

- IAM Role attached?
- Policy allows `s3:GetObject`?
- Bucket Policy?
- KMS permissions (if encrypted)?

---

## Scenario 2

### Problem

Developer cannot launch EC2.

Check:

- IAM Group
- Attached Policies
- Explicit Deny
- Service Control Policies (if using AWS Organizations)

---

## Scenario 3

### Problem

Lambda receives AccessDenied.

Check:

- Execution Role
- IAM Policy
- Trust Policy
- Resource Policy

---

## Scenario 4

### Problem

Access Key leaked to GitHub.

Immediate actions:

- Disable the key
- Delete the key
- Create a new key if required
- Review CloudTrail for misuse
- Rotate any affected credentials

---

# 23. Interview Questions

## Question 1

What is IAM?

### Answer

IAM is AWS Identity and Access Management. It controls authentication and authorization for AWS resources.

---

## Question 2

Difference between IAM User and IAM Role?

### Answer

An IAM User is a permanent identity with long-term credentials. An IAM Role provides temporary credentials and is assumed by users, AWS services, or applications.

---

## Question 3

Why are Roles preferred for EC2?

### Answer

Roles provide temporary credentials that AWS automatically rotates, eliminating the need to store long-term access keys on the instance.

---

## Question 4

What is the Principle of Least Privilege?

### Answer

Grant only the minimum permissions necessary to perform a task, reducing the risk of accidental or malicious actions.

---

## Question 5

Difference between Managed Policy and Inline Policy?

### Answer

Managed Policies are reusable and can be attached to multiple identities. Inline Policies are embedded into a single identity and cannot be reused.

---

## Question 6

Why should you avoid using the Root User?

### Answer

The Root User has unrestricted permissions. Compromise of the Root account can affect the entire AWS account. It should be protected with MFA and reserved for rare administrative tasks.

---

## Question 7

Can an IAM User belong to multiple Groups?

### Answer

Yes.

A user can be a member of multiple IAM Groups, and permissions are combined.

---

## Question 8

Can a Group contain another Group?

### Answer

No.

IAM Groups cannot be nested.

---

# 24. Amazon Follow-up Questions

### Question

Can an IAM Role have Access Keys?

### Answer

No.

Roles use temporary security credentials obtained through AWS Security Token Service (STS), not long-term access keys.

---

### Question

Can an IAM User assume a Role?

### Answer

Yes.

If the trust policy and IAM permissions allow it, an IAM User can assume an IAM Role.

---

### Question

Can a Policy exist without being attached?

### Answer

Yes.

Customer Managed Policies can exist independently until attached to users, groups, or roles.

---

### Question

Can IAM control resources in another AWS account?

### Answer

Yes, through cross-account role assumption and appropriate trust policies, which we'll cover in Part 3.

---

# 25. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Create:

- IAM User
- IAM Group

Add the user to the group.

---

## Lab 2

Attach the `AmazonS3ReadOnlyAccess` policy to the group.

Verify S3 access.

---

## Lab 3

Create an IAM Role for EC2.

Attach an S3 read-only policy.

Launch an EC2 instance with the role and verify S3 access without configuring access keys.

---

## Lab 4

Create a Customer Managed Policy that allows:

- Start EC2
- Stop EC2

Attach it to a test user.

Verify allowed and denied operations.

---

## Lab 5

Enable MFA for an IAM User and verify login behavior.

---

# 26. One-Page Revision

```
Authentication
      │
      ▼
 IAM User / IAM Role
      │
      ▼
    Policy
      │
      ▼
AWS Service
```

Remember:

- IAM = Authentication + Authorization
- User = Permanent identity
- Group = Collection of users
- Role = Temporary identity
- Policy = Permissions
- Root User = Avoid daily use
- Least Privilege = Security best practice
- Roles > Access Keys for applications

---

# Think Like a Production Engineer

When you see an **AccessDenied** error, don't guess.

Follow this sequence:

1. Who is making the request? (User or Role)
2. Which policy is attached?
3. Is the required action allowed?
4. Is there an explicit deny?
5. Is there a resource policy involved?
6. Is there an SCP or permission boundary restricting access?

This structured approach helps identify IAM authorization issues quickly and accurately.

# End of Part 1
