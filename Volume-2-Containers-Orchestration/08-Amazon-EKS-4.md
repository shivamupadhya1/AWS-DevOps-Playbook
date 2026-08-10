# Amazon Elastic Kubernetes Service (EKS)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 08
>
> Part 4 – Security (IAM, IRSA, RBAC, Secrets & Network Security)

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand EKS security architecture
- Explain IAM vs RBAC
- Explain Service Accounts
- Master IRSA
- Secure applications using AWS Secrets Manager
- Understand Kubernetes Secrets
- Understand Network Policies
- Understand Security Groups
- Debug IAM-related production issues
- Answer advanced interview questions

---

# 1. Security Layers in EKS

Security in EKS is implemented at multiple layers.

```
IAM

↓

RBAC

↓

Service Account

↓

IRSA

↓

Pod

↓

Application
```

Each layer has a different responsibility.

---

# 2. AWS IAM

IAM controls access to AWS resources.

Examples

- S3
- DynamoDB
- SQS
- SNS
- Secrets Manager
- KMS

IAM **does not** control Kubernetes permissions.

---

# 3. Kubernetes RBAC

RBAC controls what users or service accounts can do **inside the Kubernetes cluster**.

Examples

- Create Pods
- Delete Deployments
- Read ConfigMaps
- Update Services
- View Secrets

RBAC **does not** grant access to AWS services.

---

# 4. IAM vs RBAC

| Feature | IAM | RBAC |
|---------|------|------|
| Controls AWS Resources | ✅ | ❌ |
| Controls Kubernetes API | ❌ | ✅ |
| S3 Access | ✅ | ❌ |
| Pod Creation | ❌ | ✅ |
| Secrets Manager Access | ✅ | ❌ |
| Namespace Permissions | ❌ | ✅ |

Interviewers ask this very often.

---

# 5. Service Account

Every Pod runs using a Service Account.

Default

```
default
```

Production

```
payment-sa

inventory-sa

order-sa
```

Avoid using the default Service Account for production workloads.

---

# 6. Why Custom Service Accounts?

Benefits

- Better isolation
- Easier auditing
- Separate permissions
- Required for IRSA

---

# 7. What is IRSA?

⭐⭐⭐⭐⭐

IRSA stands for

```
IAM Roles for Service Accounts
```

It allows a Kubernetes Service Account to assume an IAM Role.

Instead of giving permissions to the entire worker node, permissions are granted only to the Pods that need them.

---

# 8. Why IRSA?

Without IRSA

```
Worker Node IAM Role

↓

All Pods

↓

Same Permissions
```

With IRSA

```
Service Account

↓

IAM Role

↓

Specific Pod

↓

Least Privilege
```

---

# 9. IRSA Architecture

```
Pod

↓

Service Account

↓

OIDC Provider

↓

IAM Role

↓

Temporary AWS Credentials

↓

AWS Service
```

---

# 10. OIDC Provider

EKS uses an OpenID Connect (OIDC) identity provider.

Flow

```
Pod

↓

Service Account Token

↓

OIDC

↓

STS AssumeRoleWithWebIdentity

↓

Temporary Credentials
```

No long-lived AWS credentials are stored inside the Pod.

---

# 11. Why Temporary Credentials?

Benefits

- Automatically rotated
- Short-lived
- More secure
- No access keys in containers

---

# 12. Worker Node IAM Role

Worker Nodes require an IAM Role for:

- Joining the cluster
- Pulling images
- Networking
- Node management

It should **not** be used for application permissions.

---

# 13. Worker Node IAM Role vs IRSA

| Feature | Worker Node IAM Role | IRSA |
|----------|----------------------|------|
| Scope | Entire Node | Individual Pod |
| Security | Lower | Higher |
| Least Privilege | Difficult | Easy |
| Fine-Grained Access | ❌ | ✅ |
| Recommended for App Access | ❌ | ✅ |

---

# 14. Example

Application

```
Payment Service
```

Needs

```
Read Secrets

Write SQS
```

Create

- IAM Role
- Service Account
- IRSA mapping

Only that Pod receives the permissions.

---

# 15. Kubernetes Secrets

Used for

- Passwords
- Tokens
- Certificates

Remember:

Kubernetes Secrets are **Base64 encoded**, not encrypted by default.

Enable etcd encryption for stronger protection.

---

# 16. Secrets Manager vs Kubernetes Secrets

| Feature | Kubernetes Secret | AWS Secrets Manager |
|----------|-------------------|---------------------|
| Encryption by Default | No (Base64 only) | Yes (KMS) |
| Automatic Rotation | ❌ | ✅ |
| Cross-Service Access | ❌ | ✅ |
| Audit | Limited | CloudTrail |
| Recommended for Production | Limited | Yes |

---

# 17. ConfigMap vs Secret

| ConfigMap | Secret |
|------------|--------|
| Non-sensitive | Sensitive |
| Plain configuration | Passwords, tokens |
| Feature flags | API keys |
| Application properties | Database credentials |

---

# 18. KMS Encryption

AWS KMS encrypts:

- Secrets Manager secrets
- EBS volumes
- S3 objects
- EKS Secret encryption (optional)

Use customer-managed keys (CMKs) when compliance requires key control.

---

# 19. Network Policies

Network Policies control Pod-to-Pod communication.

Example

```
Frontend

↓

Allowed

↓

Backend

Database

↓

Blocked
```

They implement east-west traffic controls within the cluster.

---

# 20. Security Groups

Typical Production

```
Internet

↓

ALB

↓

Security Group

↓

Worker Nodes / Pods

↓

Security Group

↓

Database
```

Allow only required ports.

---

# 21. Security Groups for Pods

With supported configurations, Pods can receive dedicated Security Groups.

Benefits

- Fine-grained network isolation
- Different security rules for different applications
- Reduced blast radius

---

# 22. Production Scenario 1

Problem

Application gets

```
AccessDenied

S3
```

Investigation

- Service Account
- IRSA annotation
- IAM Role
- IAM Policy
- Bucket Policy

---

# 23. Production Scenario 2

Problem

All Pods suddenly have S3 access.

Root Cause

Permissions granted to the Worker Node IAM Role instead of using IRSA.

---

# 24. Production Scenario 3

Problem

IRSA configured.

Still getting AccessDenied.

Investigation

- IAM Role trust policy
- OIDC provider
- Service Account annotation
- Namespace
- IAM policy permissions
- STS endpoint connectivity (if applicable)

---

# 25. Production Scenario 4

Problem

Secret not available inside Pod.

Investigation

- Secret exists
- Namespace
- Volume mount or environment variable
- Service Account
- IAM Role
- Secrets Manager permissions

---

# 26. Production Scenario 5

Problem

Frontend reaches the database directly.

Requirement

Only backend should access the database.

Solution

Network Policies and Security Groups.

---

# 27. Best Practices

- Use IRSA for application access.
- Keep Worker Node IAM Roles minimal.
- Use dedicated Service Accounts.
- Store secrets in AWS Secrets Manager.
- Encrypt sensitive data with KMS.
- Enable CloudTrail for auditing.
- Avoid wildcard IAM permissions.
- Use Network Policies where supported.

---

# 28. Common Mistakes

❌ Giving S3 permissions to the Worker Node IAM Role.

---

❌ Running everything with the default Service Account.

---

❌ Storing AWS access keys inside containers.

---

❌ Using `AdministratorAccess` for applications.

---

❌ Confusing IAM with RBAC.

---

# 29. Interview Questions

## Question 1

What is IRSA?

### Perfect Answer

IRSA (IAM Roles for Service Accounts) allows a Kubernetes Service Account to assume an IAM Role. This enables Pods to access AWS services using temporary credentials without sharing the Worker Node IAM Role.

---

## Question 2

Why is IRSA better than using the Worker Node IAM Role?

### Perfect Answer

The Worker Node IAM Role grants permissions to all Pods on that node. IRSA provides fine-grained, Pod-level permissions and follows the principle of least privilege, improving security.

---

## Question 3

Difference between IAM and RBAC?

### Perfect Answer

IAM controls access to AWS resources such as S3, DynamoDB, or Secrets Manager. RBAC controls access to Kubernetes resources such as Pods, Deployments, and Services.

---

## Question 4

Why shouldn't applications use the Worker Node IAM Role?

### Perfect Answer

Because every Pod running on the node would inherit the same permissions, increasing the attack surface. IRSA isolates permissions to only the Pods that require them.

---

## Question 5

Why use AWS Secrets Manager instead of Kubernetes Secrets?

### Perfect Answer

Secrets Manager provides KMS encryption, automatic rotation, CloudTrail auditing, and secure integration with AWS services. Kubernetes Secrets are only Base64 encoded unless additional encryption is configured.

---

## Question 6

What is the purpose of a Service Account?

### Perfect Answer

A Service Account provides an identity for Pods inside Kubernetes. With IRSA, it is also used to associate a Pod with an IAM Role.

---

## Question 7

Can IRSA work without an OIDC provider?

### Perfect Answer

No.

IRSA depends on the cluster's OIDC provider to exchange the Kubernetes Service Account token for temporary AWS credentials through AWS STS.

---

## Question 8

How does a Pod securely access S3?

### Perfect Answer

The Pod uses a Service Account associated with an IAM Role through IRSA. AWS STS issues temporary credentials, and the application uses those credentials to access S3.

---

# 30. Amazon Cross Questions

### Question

Your application suddenly receives AccessDenied for Secrets Manager after a deployment. What do you check first?

### Perfect Answer

I verify the Service Account in use, confirm the IRSA annotation, inspect the IAM Role and attached policies, ensure the OIDC trust relationship is correct, and check CloudTrail for denied API calls.

---

### Question

Can two Pods in the same namespace have different AWS permissions?

### Perfect Answer

Yes.

Each Pod can use a different Service Account mapped to a different IAM Role through IRSA.

---

### Question

Would you ever give AdministratorAccess to a Pod?

### Perfect Answer

No.

Pods should receive only the minimum permissions required for their specific workload.

---

### Question

Can RBAC grant S3 permissions?

### Perfect Answer

No.

RBAC controls Kubernetes resources only. AWS permissions must be granted through IAM, typically using IRSA.

---

# 31. Hands-on Labs (To Perform Later)

## Lab 1

Enable the cluster OIDC provider.

---

## Lab 2

Create an IAM Role for S3 read access.

---

## Lab 3

Create a Kubernetes Service Account.

---

## Lab 4

Associate the Service Account with the IAM Role (IRSA).

---

## Lab 5

Deploy a Pod using the Service Account.

---

## Lab 6

Verify the Pod can access S3.

---

## Lab 7

Remove the IAM permission and verify the application receives an `AccessDenied` error.

---

# 32. One-Page Revision

```
                IAM
                 │
         IAM Role (IRSA)
                 │
          Service Account
                 │
                Pod
                 │
          Temporary STS Credentials
                 │
           S3 / SQS / DynamoDB

RBAC
 │
Pods
Deployments
Services
ConfigMaps
```

Remember

- IAM
- RBAC
- Service Accounts
- IRSA
- OIDC
- STS
- Worker Node IAM Role
- Secrets Manager
- KMS
- Network Policies

---

# Think Like a Production Engineer

Don't think:

> "The Pod needs S3 access."

Think:

> "Which identity is the Pod using?"

Troubleshooting flow:

```
AccessDenied
     │
Which Service Account?
     │
IRSA Annotation?
     │
IAM Role?
     │
Trust Policy?
     │
IAM Permissions?
     │
CloudTrail Logs?
```

For Kubernetes API issues:

```
Permission Denied
      │
RBAC?
      │
Role?
      │
RoleBinding?
      │
ClusterRole?
      │
ClusterRoleBinding?
```

Always determine **whether the problem is AWS authorization (IAM/IRSA)** or **Kubernetes authorization (RBAC)** before troubleshooting.

# End of Part 4
