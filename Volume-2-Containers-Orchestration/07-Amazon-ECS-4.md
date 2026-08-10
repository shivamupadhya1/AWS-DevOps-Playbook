# Amazon ECS (Elastic Container Service)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 07
>
> Part 4 – Security, IAM & Secrets

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand ECS security architecture
- Explain Task Role vs Task Execution Role
- Secure applications using IAM
- Integrate Secrets Manager
- Integrate Parameter Store
- Understand ECR authentication
- Apply least privilege
- Secure production ECS workloads
- Answer IAM interview questions confidently

---

# 1. ECS Security Architecture

```
                IAM
                 │
        ┌────────┴────────┐
        │                 │
Task Execution Role    Task Role
        │                 │
        ▼                 ▼
Pull Image          Access AWS Services
from ECR
```

Remember this diagram.

Nearly every interview asks this.

---

# 2. Two IAM Roles in ECS

ECS commonly uses two IAM roles.

```
Task Execution Role

Task Role
```

Most interview confusion happens here.

---

# 3. Task Execution Role

⭐⭐⭐⭐⭐

Purpose

Used by **ECS itself** before your application starts.

Responsibilities

- Pull image from ECR
- Send logs to CloudWatch
- Retrieve Secrets Manager values
- Retrieve Parameter Store values

Think of it as:

> Permissions used to **start the container**.

---

# 4. Task Role

⭐⭐⭐⭐⭐

Purpose

Used by the **application running inside the container**.

Examples

Application wants to

- Read S3
- Write DynamoDB
- Publish SNS
- Read SQS
- Access Secrets Manager
- Call AWS APIs

Those permissions belong in the Task Role.

---

# 5. Easy Way to Remember

Task Execution Role

↓

Container startup

Task Role

↓

Application runtime

---

# 6. Example

Application

```
Spring Boot
```

Needs

```
S3

DynamoDB

SNS
```

Assign permissions

```
Task Role
```

NOT

Execution Role.

---

# 7. ECR Authentication Flow

```
Task Starts

↓

Execution Role

↓

ECR Authentication Token

↓

Pull Image

↓

Container Starts

↓

Task Role Active

↓

Application Calls AWS
```

This flow is frequently asked.

---

# 8. Why Separate Roles?

Suppose

Application only needs

```
S3 Read
```

It should not also receive

```
ECR Push

CloudWatch Admin

Secrets Admin
```

Separation follows the **Principle of Least Privilege**.

---

# 9. Principle of Least Privilege

⭐⭐⭐⭐⭐

Grant only required permissions.

Example

Instead of

```
AmazonS3FullAccess
```

Use

```
s3:GetObject

s3:ListBucket
```

Only.

---

# 10. Secrets Management

Never store

```
Password

API Key

Token
```

inside

- Dockerfile
- Git
- Environment files
- Source code

Use

```
AWS Secrets Manager
```

---

# 11. Secrets Manager Integration

Flow

```
Secrets Manager

↓

Execution Role retrieves secret

↓

Injected into Container

↓

Application uses secret
```

Secrets remain outside the image.

---

# 12. Parameter Store

Suitable for

- Configuration
- URLs
- Feature flags
- Non-sensitive values

SecureString parameters can also be encrypted using KMS.

---

# 13. Secrets Manager vs Parameter Store

| Feature | Secrets Manager | Parameter Store |
|----------|-----------------|-----------------|
| Passwords | ✅ | ✅ (SecureString) |
| Automatic Rotation | ✅ | ❌ |
| KMS Encryption | ✅ | ✅ |
| API Keys | ✅ | ✅ |
| Configuration Values | Possible | Excellent |

---

# 14. Environment Variables

Instead of

```
DB_PASSWORD=password123
```

Use

```
Secret ARN
```

in the Task Definition.

ECS resolves the secret securely.

---

# 15. IAM Policy Example

Application needs

```
Read

↓

S3 Bucket
```

Policy

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject"
  ],
  "Resource": [
    "arn:aws:s3:::my-bucket/*"
  ]
}
```

Avoid wildcard permissions whenever possible.

---

# 16. Security Groups

Typical Production

```
Internet

↓

ALB

↓

Security Group

↓

ECS Tasks

↓

Security Group

↓

Database
```

Do not expose Tasks directly to the Internet.

---

# 17. Private Subnets

Best Practice

```
ALB

↓

Public Subnet

↓

ECS Tasks

↓

Private Subnet

↓

Database

↓

Private Subnet
```

This limits direct access.

---

# 18. ECR Security

Recommendations

- Private repositories
- IAM authentication
- Scan-on-push
- Lifecycle policies
- KMS encryption

---

# 19. Logging Security

Send logs to

```
CloudWatch Logs
```

Benefits

- Audit trail
- Incident investigation
- Centralized logging

---

# 20. Production Security Checklist

✔ Use private subnets

✔ Use ALB

✔ Use Security Groups

✔ Use Task Roles

✔ Use Secrets Manager

✔ Use CloudWatch Logs

✔ Enable ECR scanning

✔ Use immutable image tags

✔ Keep images updated

✔ Follow least privilege

---

# 21. Production Scenario 1

Problem

Application cannot access S3.

Investigation

```
Task Role

↓

IAM Policy

↓

Bucket Policy
```

Root Cause

Task Role missing permission.

---

# 22. Production Scenario 2

Problem

Task cannot pull image.

Investigation

```
Execution Role

↓

ECR Permission

↓

Repository Access
```

Root Cause

Execution Role missing ECR permissions.

---

# 23. Production Scenario 3

Problem

Secret retrieval fails.

Investigation

- Secret ARN
- Region
- IAM permissions
- KMS permissions

---

# 24. Production Scenario 4

Problem

Developer stored AWS credentials in Dockerfile.

Risk

Anyone with the image can extract them.

Correct Solution

Use IAM Task Roles.

---

# 25. Best Practices

- Never store AWS credentials inside containers.
- Use Task Roles.
- Use Execution Roles only for startup operations.
- Store secrets externally.
- Enable logging.
- Keep Tasks in private subnets.
- Grant minimum permissions.

---

# 26. Common Mistakes

❌ Giving S3 permissions to the Execution Role instead of the Task Role.

---

❌ Embedding passwords in Docker images.

---

❌ Using `AdministratorAccess`.

---

❌ Using wildcard (`*`) permissions everywhere.

---

❌ Exposing ECS Tasks directly to the Internet.

---

# 27. Interview Questions

## Question 1

What is the difference between Task Role and Task Execution Role?

### Perfect Answer

The Task Execution Role is used by ECS during container startup to pull images from ECR, retrieve secrets, and send logs to CloudWatch. The Task Role is assumed by the running application and grants permissions to access AWS services such as S3, DynamoDB, or SQS.

---

## Question 2

Which role is used to pull images from ECR?

### Perfect Answer

The Task Execution Role.

ECS uses this role to authenticate with ECR and pull container images before the application starts.

---

## Question 3

Which role allows a Spring Boot application to read from S3?

### Perfect Answer

The Task Role because the application accesses AWS services after the container has started.

---

## Question 4

Why should AWS credentials never be stored in Docker images?

### Perfect Answer

Anyone with access to the image could extract those credentials. IAM Task Roles provide temporary credentials securely without embedding secrets.

---

## Question 5

Why use Secrets Manager?

### Perfect Answer

Secrets Manager securely stores sensitive information, supports encryption with KMS, enables automatic rotation, and integrates directly with ECS Task Definitions.

---

## Question 6

What is the Principle of Least Privilege?

### Perfect Answer

It means granting only the permissions required for a specific task and nothing more, reducing the impact of compromised applications or accidental misuse.

---

## Question 7

Can Task Roles and Execution Roles be the same IAM Role?

### Perfect Answer

Technically they can be, but it is not recommended. Separating them follows the principle of least privilege and reduces security risk.

---

## Question 8

How does ECS obtain secrets securely?

### Perfect Answer

The Task Definition references the secret ARN. During startup, ECS retrieves the secret using the appropriate IAM permissions and injects it into the container without storing it in the image.

---

## Question 9

Why place ECS Tasks in private subnets?

### Perfect Answer

Private subnets prevent direct Internet access to application containers. External traffic should flow through an Application Load Balancer, improving security.

---

## Question 10

How would you secure an ECS production environment?

### Perfect Answer

I would use private ECR repositories, immutable image tags, scan-on-push, private subnets, ALB, Security Groups, Task Roles, Secrets Manager, CloudWatch logging, and least-privilege IAM policies.

---

# 28. Amazon Cross Questions

### Question

The application cannot read S3. Which IAM role do you check first?

### Perfect Answer

The Task Role because it controls permissions used by the running application.

---

### Question

The image cannot be pulled from ECR. Which IAM role do you check?

### Perfect Answer

The Task Execution Role because ECS uses it to authenticate with ECR before the container starts.

---

### Question

Would you give AdministratorAccess to a Task Role?

### Perfect Answer

No.

I would grant only the permissions required by the application according to the principle of least privilege.

---

### Question

Where should database passwords be stored?

### Perfect Answer

In AWS Secrets Manager, referenced from the ECS Task Definition, rather than inside the Docker image or source code.

---

# 29. Hands-on Labs (To Perform Later)

## Lab 1

Create a Task Execution Role with ECR and CloudWatch permissions.

---

## Lab 2

Create a Task Role with read-only S3 access.

---

## Lab 3

Store a database password in Secrets Manager.

---

## Lab 4

Reference the secret in an ECS Task Definition.

---

## Lab 5

Deploy an application that reads the secret successfully.

---

## Lab 6

Remove the Task Role permission and observe the AccessDenied error.

---

# 30. One-Page Revision

```
          ECS Security

                │
     ┌──────────┴──────────┐
     │                     │
Task Execution Role    Task Role
     │                     │
ECR, Logs, Secrets     Application
     │                     │
Pull Image           S3, SQS, DynamoDB

        Secrets Manager
               │
         Secure Injection

        Private Subnets
               │
              ALB
```

Remember

- Task Role
- Task Execution Role
- Secrets Manager
- Parameter Store
- Least Privilege
- Private Subnets
- Security Groups
- ECR Authentication
- CloudWatch Logs

---

# 31. Think Like a Production Engineer

Don't think:

> "The application needs AWS access."

Think:

> "Which identity should perform this action?"

Decision Flow

```
Need to pull image?

↓

Task Execution Role

Need to access AWS service?

↓

Task Role

Need a password?

↓

Secrets Manager

Need configuration?

↓

Parameter Store
```

The easiest interview shortcut to remember:

> **Execution Role = Start the container**
>
> **Task Role = Run the application**

# End of Part 4
