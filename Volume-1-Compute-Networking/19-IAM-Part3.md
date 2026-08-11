# IAM – Part 3 (STS, AssumeRole, Federation & IAM Identity Center)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 19
>
> IAM – Part 3

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand AWS STS
- Understand Temporary Credentials
- Understand AssumeRole
- Understand Trust Policies
- Understand Cross-Account Authentication
- Understand External ID
- Understand Web Identity Federation
- Understand SAML Federation
- Understand IAM Identity Center (AWS SSO)
- Design Enterprise Authentication

---

# 1. Why AWS Created STS

Imagine a company with:

- 500 Developers
- 200 DevOps Engineers
- 100 Applications
- Hundreds of AWS Accounts

Would AWS create permanent Access Keys for everyone?

No.

Instead AWS created:

```
Security Token Service

(STS)
```

STS provides:

```
Temporary Credentials
```

instead of permanent credentials.

---

# 2. Long-Term vs Temporary Credentials

Long-Term

```
Access Key

Secret Key

Never Expires
```

Problems

- Can leak
- Hard to rotate
- Stored in applications

---

Temporary

```
Access Key

Secret Key

Session Token

Expires Automatically
```

Much safer.

---

# 3. What is STS?

AWS Security Token Service issues temporary credentials.

Example

```
Developer

↓

STS

↓

Temporary Credentials

↓

AWS Resources
```

These credentials automatically expire.

---

# 4. Temporary Credential Components

STS returns:

```
Access Key

Secret Key

Session Token

Expiration Time
```

All four are required.

---

# 5. AssumeRole

The most commonly used STS operation.

```
User

↓

AssumeRole

↓

Temporary Credentials

↓

AWS
```

The caller temporarily becomes the target role.

---

# 6. Why AssumeRole?

Instead of giving everyone:

```
AdministratorAccess
```

Create a role.

Users assume it only when needed.

Benefits

- Better Security
- Temporary Permissions
- Auditable
- Easy Revocation

---

# 7. AssumeRole Flow

```
Developer

↓

IAM User

↓

STS AssumeRole

↓

Temporary Credentials

↓

Production Account
```

---

# 8. Trust Policy

Every IAM Role contains a Trust Policy.

Question:

```
Who is allowed to assume me?
```

Example

```json
{
 "Version":"2012-10-17",
 "Statement":[
   {
      "Effect":"Allow",
      "Principal":{
          "AWS":"arn:aws:iam::111111111111:root"
      },
      "Action":"sts:AssumeRole"
   }
 ]
}
```

---

# 9. Identity Policy vs Trust Policy

Identity Policy

```
What am I allowed to do?
```

Trust Policy

```
Who may assume this role?
```

Very common interview question.

---

# 10. Cross-Account Access

We already covered the complete production implementation for:

```
Account A

↓

Assume Role

↓

Account B

↓

Access S3
```

Remember:

Two permissions are required:

Account A

↓

Permission to call

```
sts:AssumeRole
```

Account B

↓

Trust Policy

```
Allows Account A
```

---

# 11. External ID

Suppose

Company hires:

```
Monitoring Vendor
```

Vendor manages

100 customers.

How does AWS prevent the confused deputy problem?

Answer

```
External ID
```

The vendor must provide the expected External ID when assuming the role.

---

# 12. Why External ID Exists

Without External ID

```
Customer A

↓

Vendor

↓

Wrong Role
```

Possible confusion.

With External ID

AWS verifies

```
External ID Matches?
```

If not

↓

Access Denied.

---

# 13. STS APIs

Important APIs

```
AssumeRole

AssumeRoleWithSAML

AssumeRoleWithWebIdentity

GetCallerIdentity

GetSessionToken
```

Know these names for interviews.

---

# 14. GetCallerIdentity

Very useful command.

```
aws sts get-caller-identity
```

Returns

- Account ID
- User / Role ARN
- Caller Identity

Useful while debugging CI/CD pipelines.

---

# 15. Federation

Federation means

```
Login Somewhere Else

↓

Access AWS
```

Instead of creating IAM Users.

---

# 16. SAML Federation

Example

```
Active Directory

↓

Okta

↓

Azure AD

↓

AWS
```

User logs in once.

AWS trusts the identity provider.

---

# 17. Web Identity Federation

Used for

- GitHub Actions
- Google Login
- Facebook Login
- Kubernetes Service Accounts (IRSA)
- Mobile Apps

Authentication comes from an external identity provider.

---

# 18. GitHub Actions Example

Instead of storing:

```
AWS Access Key

Secret Key
```

GitHub requests an OIDC token.

AWS verifies it.

STS returns temporary credentials.

Benefits:

- No long-term secrets
- Automatic expiration
- Better security

---

# 19. EKS IRSA (IAM Roles for Service Accounts)

Without IRSA

```
All Pods

↓

Node IAM Role
```

Every pod shares the node's permissions.

---

With IRSA

```
Pod

↓

Service Account

↓

IAM Role

↓

STS

↓

Temporary Credentials
```

Each workload gets only the permissions it needs.

---

# 20. IAM Identity Center (AWS SSO)

Old approach

```
500 Employees

↓

500 IAM Users
```

Hard to manage.

---

Modern approach

```
Azure AD

↓

IAM Identity Center

↓

AWS Accounts
```

Centralized login and permission management.

---

# 21. Benefits of IAM Identity Center

- Single Sign-On
- MFA Integration
- Central User Management
- Multi-Account Access
- Permission Sets
- Better Auditing

---

# 22. Permission Sets

Permission Sets are used by IAM Identity Center.

Think of them as reusable IAM permission templates.

Example

```
DevOps

↓

EC2

EKS

CloudWatch

S3
```

Assign the Permission Set to users or groups across multiple AWS accounts.

---

# 23. Production Architecture

```
Developer

↓

Azure AD / Okta

↓

IAM Identity Center

↓

Permission Set

↓

AWS Account

↓

Assume Role

↓

STS

↓

Temporary Credentials
```

---

# 24. Common Mistakes

❌ Hardcoding Access Keys in Jenkins

❌ Long-term credentials in GitHub Actions

❌ Missing Trust Policy

❌ Forgetting `sts:AssumeRole`

❌ No MFA for privileged roles

❌ Sharing IAM Users across teams

---

# 25. Best Practices

- Prefer IAM Roles over Access Keys.
- Use STS temporary credentials.
- Use IAM Identity Center for workforce access.
- Use OIDC for CI/CD systems.
- Require MFA for privileged roles.
- Use External IDs for third-party vendors.
- Regularly review role trust policies.

---

# 26. Production Scenarios

## Scenario 1

### Problem

Jenkins cannot deploy to another AWS account.

Check:

- Does Jenkins have permission for `sts:AssumeRole`?
- Does the target role trust Jenkins' account or role?
- Is the role ARN correct?
- Are there SCP restrictions?

---

## Scenario 2

### Problem

GitHub Actions deployment fails.

Check:

- OIDC provider configured?
- IAM Role trust policy?
- Repository and branch conditions?
- Workflow permissions?

---

## Scenario 3

### Problem

EKS Pod receives AccessDenied.

Check:

- IRSA configured?
- Service Account annotation?
- IAM Role trust policy?
- IAM permissions?

---

## Scenario 4

### Problem

AssumeRole succeeds but S3 access fails.

Check:

- Role permissions
- Bucket Policy
- KMS Key Policy
- Permission Boundary
- SCP

---

# 27. Interview Questions

## Question 1

What is AWS STS?

### Answer

AWS Security Token Service (STS) issues temporary security credentials that applications and users can use to access AWS resources without long-term access keys.

---

## Question 2

Why are temporary credentials preferred?

### Answer

They expire automatically, reduce the risk of credential leakage, and eliminate the need to store permanent secrets.

---

## Question 3

What is AssumeRole?

### Answer

AssumeRole is an STS operation that allows a trusted principal to temporarily use the permissions of an IAM Role.

---

## Question 4

What is a Trust Policy?

### Answer

A Trust Policy defines **who is allowed to assume an IAM Role**.

---

## Question 5

Difference between Trust Policy and Permission Policy?

### Answer

Trust Policy controls **who can assume the role**. Permission Policy controls **what actions the role can perform after it has been assumed**.

---

## Question 6

What is External ID?

### Answer

An External ID is a unique value used when third parties assume roles to help protect against the confused deputy problem.

---

## Question 7

Why use IAM Identity Center?

### Answer

It provides centralized workforce access, Single Sign-On, permission management across multiple AWS accounts, and integrates with external identity providers.

---

## Question 8

How does GitHub Actions access AWS securely?

### Answer

Using OpenID Connect (OIDC). GitHub obtains an identity token, AWS validates it, and STS issues temporary credentials. No long-term AWS access keys are required.

---

# 28. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Run:

```
aws sts get-caller-identity
```

Observe:

- Account ID
- ARN
- User/Role

---

## Lab 2

Create an IAM Role and allow a test IAM User to assume it.

Use:

```
aws sts assume-role
```

Verify temporary credentials.

---

## Lab 3

Configure cross-account role assumption between two AWS accounts.

(Reuse the complete S3 cross-account setup from the earlier playbook chapter.)

---

## Lab 4

Configure GitHub Actions with OIDC authentication to deploy infrastructure without storing AWS access keys.

---

## Lab 5

Create an IAM Identity Center Permission Set and assign it to a test user for a development AWS account.

---

# 29. One-Page Revision

```
User / App
     │
     ▼
    STS
     │
Temporary Credentials
     │
 AssumeRole
     │
 IAM Role
     │
 AWS Resources
```

Remember:

- STS = Temporary credentials
- AssumeRole = Temporary role assumption
- Trust Policy = Who can assume
- Permission Policy = What the role can do
- External ID = Third-party security
- OIDC = GitHub Actions / Kubernetes
- IAM Identity Center = Enterprise SSO

---

# Think Like a Production Engineer

Whenever you need access across accounts or from automation:

- Don't create long-term IAM users.
- Create an IAM Role.
- Configure the correct Trust Policy.
- Grant only the required permissions.
- Use STS to obtain temporary credentials.
- Audit role assumptions with CloudTrail.

This is the standard pattern used in modern AWS production environments.

# End of Part 3
