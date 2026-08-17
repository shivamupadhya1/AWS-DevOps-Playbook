# IAM – Part 4 (Production, Security & Troubleshooting)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 20
>
> IAM – Part 4

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Troubleshoot IAM AccessDenied errors
- Secure IAM in production
- Handle credential leaks
- Audit IAM permissions
- Use IAM Access Analyzer
- Generate IAM Credential Reports
- Debug IAM issues in Jenkins, Terraform, Lambda, ECS and EKS
- Answer advanced interview scenarios

---

# 1. IAM in Production

Every AWS request follows this path:

```
Application
      │
IAM Role / IAM User
      │
IAM Policy
      │
Resource Policy
      │
Permission Boundary
      │
Session Policy
      │
SCP
      │
AWS Service
```

If any layer blocks access:

```
AccessDenied
```

---

# 2. Production Troubleshooting Strategy

Never start by changing permissions.

Instead verify:

```
Who is making the request?

↓

Which credentials are being used?

↓

Which IAM Role/User?

↓

What API is failing?

↓

Which policy blocks it?
```

---

# 3. AccessDenied Checklist

Whenever you see:

```
AccessDenied
```

Check:

```
✓ IAM Role

✓ IAM Policy

✓ Resource Policy

✓ Trust Policy

✓ Permission Boundary

✓ Session Policy

✓ SCP

✓ KMS Key Policy

✓ Region

✓ Correct ARN
```

---

# 4. Scenario 1

## Jenkins Pipeline Fails

Error

```
AccessDenied

ec2:RunInstances
```

Investigation

```
Jenkins

↓

IAM Role

↓

Policy

↓

RunInstances Allowed?
```

If not

↓

Update policy.

---

# 5. Scenario 2

Terraform Cannot Create S3 Bucket

Possible causes

- Missing `s3:CreateBucket`
- SCP restriction
- Bucket already exists
- Wrong Region
- Permission Boundary

---

# 6. Scenario 3

Lambda Cannot Read Secrets Manager

Check:

- Execution Role
- `secretsmanager:GetSecretValue`
- KMS permissions
- Secret resource policy

---

# 7. Scenario 4

EKS Pod Gets AccessDenied

Check:

- IRSA configured?
- Service Account annotation?
- IAM Role permissions?
- Trust policy?
- OIDC provider?

---

# 8. Scenario 5

CodeBuild Cannot Push Docker Image to ECR

Verify:

```
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:PutImage
ecr:InitiateLayerUpload
ecr:UploadLayerPart
ecr:CompleteLayerUpload
```

---

# 9. Scenario 6

CloudFormation Stack Fails

Possible causes

- Service Role missing
- IAM permissions
- SCP
- Stack execution role
- Resource policy

---

# 10. Scenario 7

Cross-Account Access Stops Working

Check:

- Trust Policy
- AssumeRole permission
- Bucket Policy
- KMS Key Policy
- External ID (if used)

---

# 11. Scenario 8

Developer Has AdministratorAccess But Still Gets Denied

Possible causes

- SCP
- Permission Boundary
- Resource Policy Explicit Deny
- KMS Key Policy
- Session Policy

---

# 12. IAM Access Analyzer

IAM Access Analyzer helps identify:

- Publicly accessible resources
- Cross-account access
- Unintended sharing
- External principals

Useful for:

- S3 Buckets
- KMS Keys
- IAM Roles
- SQS
- SNS
- Secrets Manager

---

# 13. IAM Credential Report

Credential Reports provide:

- Password enabled
- MFA enabled
- Access key age
- Password age
- Last console login
- Last access key usage

Useful for security audits.

---

# 14. IAM Policy Simulator

Before deploying a policy:

Use the IAM Policy Simulator.

Benefits:

- Test Allow/Deny decisions
- Validate conditions
- Prevent production failures

---

# 15. Access Advisor

Access Advisor shows:

```
Which AWS services an IAM identity has actually used.
```

Useful for removing unused permissions and applying least privilege.

---

# 16. IAM Last Accessed Information

AWS records when an IAM identity last used many AWS services.

Use this to:

- Remove unused permissions
- Clean up old roles
- Audit inactive identities

---

# 17. Detecting Over-Permissioned Roles

Example

Role has:

```
AdministratorAccess
```

But only uses:

```
CloudWatch

S3

EC2
```

Reduce permissions to only the required services.

---

# 18. Handling Credential Leaks

Suppose an Access Key is accidentally committed to GitHub.

Immediate response:

1. Disable the key.
2. Delete the key.
3. Create a replacement if needed.
4. Review CloudTrail for suspicious activity.
5. Rotate any affected credentials.
6. Verify no additional secrets were exposed.
7. Investigate the root cause and update development practices.

---

# 19. CloudTrail for IAM

CloudTrail records IAM events such as:

- CreateUser
- DeleteUser
- AttachRolePolicy
- AssumeRole
- CreateAccessKey
- DeleteAccessKey
- PutRolePolicy

During investigations, CloudTrail is often the first place to check.

---

# 20. IAM Monitoring

Recommended monitoring:

- CloudTrail
- CloudWatch Alarms
- AWS Config
- IAM Access Analyzer
- GuardDuty
- Security Hub

---

# 21. IAM Security Best Practices

- Enable MFA for privileged users.
- Avoid Root User usage.
- Use IAM Roles instead of Access Keys.
- Rotate long-term credentials.
- Remove unused users and roles.
- Review permissions regularly.
- Follow Least Privilege.
- Use OIDC for CI/CD systems.
- Monitor CloudTrail continuously.

---

# 22. Common Production Mistakes

❌ Hardcoding Access Keys in applications.

❌ Sharing IAM Users between team members.

❌ Giving everyone AdministratorAccess.

❌ Forgetting KMS permissions.

❌ Ignoring SCPs during troubleshooting.

❌ Trusting entire AWS accounts when only one role is required.

❌ Never reviewing unused permissions.

---

# 23. Real Production Troubleshooting Flow

```
AccessDenied

↓

Who made the request?

↓

Which Role/User?

↓

IAM Policy

↓

Resource Policy

↓

Trust Policy

↓

Permission Boundary

↓

Session Policy

↓

SCP

↓

CloudTrail

↓

AWS Decision
```

---

# 24. Incident Response Checklist

If an IAM incident occurs:

```
Contain

↓

Identify

↓

Revoke

↓

Rotate

↓

Investigate

↓

Recover

↓

Document

↓

Improve Controls
```

---

# 25. Interview Questions

## Question 1

A Jenkins pipeline suddenly receives `AccessDenied`. What is your approach?

### Answer

Identify which IAM Role or credentials Jenkins is using, determine the failing API call, verify identity and resource policies, check trust policies (if assuming a role), review permission boundaries, SCPs, and CloudTrail logs before making changes.

---

## Question 2

How do you investigate an AccessDenied error?

### Answer

Start with the caller identity, review IAM policies, resource policies, trust policies, permission boundaries, session policies, SCPs, KMS key policies (if applicable), and CloudTrail logs.

---

## Question 3

What would you do if an AWS Access Key leaked?

### Answer

Disable and delete the compromised key, review CloudTrail for suspicious activity, rotate any affected credentials, assess the impact, and improve secret management practices to prevent recurrence.

---

## Question 4

How do you audit IAM permissions?

### Answer

Use IAM Access Analyzer, Credential Reports, Access Advisor, Last Accessed Information, CloudTrail, AWS Config, and periodic permission reviews.

---

## Question 5

What is IAM Access Analyzer?

### Answer

IAM Access Analyzer identifies resources that are shared externally or publicly and helps detect unintended access.

---

## Question 6

What is a Credential Report?

### Answer

A Credential Report is an account-wide report containing information about IAM users, including password status, MFA, access keys, and credential age.

---

## Question 7

How do you implement Least Privilege?

### Answer

Grant only the required actions on the required resources, remove unused permissions, review access regularly, and use tools like Access Advisor and Last Accessed Information.

---

## Question 8

What tools do you use to troubleshoot IAM?

### Answer

- IAM Policy Simulator
- CloudTrail
- IAM Access Analyzer
- Credential Reports
- Access Advisor
- AWS Config
- CloudWatch Logs (where applicable)

---

# 26. Amazon Follow-up Questions

### Question

Why might an Administrator still receive AccessDenied?

### Answer

Possible reasons include an explicit deny, an SCP restriction, a permission boundary, a session policy, or a resource policy such as a KMS key policy.

---

### Question

Can CloudTrail show who assumed a role?

### Answer

Yes.

CloudTrail records `AssumeRole` events, including the caller identity, target role, and timestamp.

---

### Question

How do you verify which identity your application is using?

### Answer

Run:

```
aws sts get-caller-identity
```

This returns the current AWS account ID and IAM ARN.

---

### Question

How do you reduce IAM permissions safely?

### Answer

Review actual usage with Access Advisor and Last Accessed Information, test reduced policies using the IAM Policy Simulator, then deploy changes gradually.

---

# 27. Hands-on Labs (When AWS Account is Ready)

## Lab 1

Use:

```
aws sts get-caller-identity
```

Confirm which role or user is active.

---

## Lab 2

Generate an IAM Credential Report.

Review:

- MFA
- Password usage
- Access key age

---

## Lab 3

Run IAM Access Analyzer and identify any externally accessible resources.

---

## Lab 4

Use the IAM Policy Simulator to troubleshoot a denied action before updating the policy.

---

## Lab 5

Review CloudTrail for recent IAM events such as:

- AssumeRole
- CreateAccessKey
- DeleteAccessKey

---

# 28. One-Page Revision

```
Application
      │
IAM Role
      │
IAM Policy
      │
Resource Policy
      │
Permission Boundary
      │
Session Policy
      │
SCP
      │
AWS Decision
```

Remember:

- Don't guess—trace the request.
- Verify the caller identity first.
- Explicit Deny overrides Allow.
- CloudTrail is your audit source.
- Access Analyzer finds unintended sharing.
- Credential Reports help with IAM audits.
- Access Advisor helps remove unused permissions.

---

# Think Like a Production Engineer

When an IAM issue occurs:

1. Identify **who** is making the request.
2. Identify **what** action is being attempted.
3. Verify **every policy layer**, not just IAM permissions.
4. Use **CloudTrail** to confirm what actually happened.
5. Fix the **root cause**, not just the immediate error.

This disciplined troubleshooting process is what distinguishes experienced DevOps engineers from engineers who rely on trial and error.

# End of Part 4
