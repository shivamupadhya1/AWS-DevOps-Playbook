# Amazon Elastic Container Registry (ECR)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 06

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand Amazon ECR architecture
- Push and pull Docker images
- Explain private and public ECR repositories
- Configure authentication
- Use lifecycle policies
- Enable vulnerability scanning
- Manage image tags
- Secure ECR using IAM
- Troubleshoot image pull failures
- Answer Amazon ECR interview questions

---

# 1. What is Amazon ECR?

Amazon Elastic Container Registry (ECR) is a fully managed Docker-compatible container image registry provided by AWS.

It stores Docker and OCI-compatible images that are later pulled by:

- Amazon ECS
- Amazon EKS
- AWS Lambda (Container Images)
- EC2
- Local Docker clients

Architecture

```
Developer

↓

Docker Build

↓

Amazon ECR

↓

ECS / EKS / Lambda
```

---

# 2. Why ECR?

Without ECR

```
Developer

↓

Docker Hub

↓

Production
```

Problems

- Rate limits
- Public exposure
- External dependency
- IAM integration unavailable

With ECR

```
Developer

↓

Amazon ECR

↓

AWS Services
```

Benefits

- Fully managed
- Highly available
- IAM integration
- Vulnerability scanning
- Lifecycle policies
- Private networking via VPC endpoints

---

# 3. ECR Architecture

```
Docker Client

↓

AWS CLI Authentication

↓

Amazon ECR

↓

Image Repository

↓

Image Layers
```

Each repository stores one or more tagged images.

---

# 4. Private vs Public ECR

## Private Repository

- Default option
- IAM protected
- Used for internal applications
- Most production workloads

## Public Repository

- Publicly accessible
- Suitable for open-source projects
- No authentication required to pull public images

---

# 5. Repository

A repository stores related images.

Example

```
backend

frontend

payment-service

inventory-service
```

Example image

```
backend:v1

backend:v2

backend:latest
```

---

# 6. Image Tags

Example

```
app:v1.0

app:v1.1

app:v2.0
```

Avoid

```
latest
```

Production Recommendation

Use versioned or immutable tags.

Examples

```
1.0.4

20260810

git-commit-sha
```

---

# 7. Authentication

ECR uses IAM authentication.

Login

```bash
aws ecr get-login-password \
| docker login \
--username AWS \
--password-stdin \
<ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com
```

Authentication token is temporary.

---

# 8. Push Image

Tag image

```bash
docker tag app:1.0 \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/app:1.0
```

Push

```bash
docker push \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/app:1.0
```

---

# 9. Pull Image

```bash
docker pull \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/app:1.0
```

ECS and EKS also pull images from ECR using IAM permissions.

---

# 10. Image Layers

ECR stores image layers efficiently.

```
Layer 1

↓

Layer 2

↓

Layer 3

↓

Layer 4
```

Shared layers are reused across image versions to reduce storage and transfer time.

---

# 11. Lifecycle Policies

⭐⭐⭐⭐⭐

Repositories can accumulate hundreds of old images.

Lifecycle policies automatically clean them.

Example

Keep only the latest 20 tagged images.

Benefits

- Reduce storage costs
- Easier maintenance
- Cleaner repositories

---

# 12. Vulnerability Scanning

⭐⭐⭐⭐⭐

ECR supports image scanning.

Scans detect:

- Critical
- High
- Medium
- Low vulnerabilities

Recommendation

Enable scan-on-push for production repositories.

---

# 13. Encryption

Images stored in ECR are encrypted at rest.

Encryption options

- AWS-managed KMS key
- Customer-managed KMS key

---

# 14. IAM Permissions

Typical permissions

```
ecr:GetAuthorizationToken

ecr:BatchGetImage

ecr:GetDownloadUrlForLayer

ecr:PutImage

ecr:InitiateLayerUpload

ecr:UploadLayerPart

ecr:CompleteLayerUpload
```

Grant only required actions.

---

# 15. Cross-Account Access

Example

Development Account

↓

Push Image

↓

Shared ECR Repository

↓

Production Account Pulls Image

Access is granted using:

- Repository policy
- IAM role
- Cross-account trust

---

# 16. Cross-Region Replication

Images can be replicated automatically.

```
Mumbai

↓

Frankfurt

↓

Virginia
```

Benefits

- Faster regional deployments
- Disaster recovery
- Reduced latency

---

# 17. Production CI/CD Flow

```
Developer

↓

GitHub

↓

Jenkins

↓

docker build

↓

Trivy Scan

↓

Amazon ECR

↓

Amazon ECS

↓

Deployment
```

---

# 18. Production Scenario 1

Problem

ECS task fails with:

```
CannotPullContainerError
```

Check

- Image exists
- Image tag correct
- Repository exists
- IAM permissions
- Network connectivity
- Authentication

---

# 19. Production Scenario 2

Problem

Repository contains 900 images.

Solution

Create a lifecycle policy.

Example

Keep only the latest 20 production images.

---

# 20. Production Scenario 3

Problem

Critical vulnerabilities detected.

Action

- Update base image
- Upgrade packages
- Rebuild image
- Scan again
- Push updated image

---

# 21. Production Scenario 4

Problem

Production deployed the wrong image.

Possible Causes

- Mutable `latest` tag
- Incorrect pipeline tag
- Manual image overwrite

Best Practice

Use immutable version tags.

---

# 22. Best Practices

- Use private repositories for production.
- Enable scan-on-push.
- Use immutable version tags.
- Apply lifecycle policies.
- Restrict IAM permissions.
- Replicate images across required regions.
- Automate image pushes from CI/CD.

---

# 23. Common Mistakes

❌ Using `latest` in production.

---

❌ Never cleaning old images.

---

❌ Granting `ecr:*` to all users.

---

❌ Ignoring vulnerability scan reports.

---

❌ Manually pushing production images from developer laptops.

---

# 24. Interview Questions

## Question 1

What is Amazon ECR?

### Perfect Answer

Amazon ECR is a fully managed container image registry that stores Docker and OCI-compatible images. It integrates with IAM and AWS container services such as ECS, EKS, and Lambda.

---

## Question 2

Why would you choose ECR over Docker Hub?

### Perfect Answer

ECR provides IAM integration, private repositories, lifecycle policies, vulnerability scanning, regional replication, and seamless integration with AWS services, making it better suited for production workloads on AWS.

---

## Question 3

What are lifecycle policies?

### Perfect Answer

Lifecycle policies automatically delete old or unneeded images based on defined rules, helping reduce storage costs and keep repositories organized.

---

## Question 4

How does ECS authenticate with ECR?

### Perfect Answer

The ECS task execution role uses IAM permissions to obtain a temporary authorization token and pull images securely from ECR.

---

## Question 5

What causes `CannotPullContainerError`?

### Perfect Answer

Common causes include incorrect image tags, missing images, insufficient IAM permissions, repository access issues, network connectivity problems, or authentication failures.

---

# 25. Amazon Cross Questions

### Question

Can two repositories contain images with the same tag?

### Perfect Answer

Yes.

Tags are unique only within a repository. Different repositories can have images tagged `v1.0` or `latest`.

---

### Question

Would you enable image scanning in production?

### Perfect Answer

Yes.

Image scanning should be enabled to identify vulnerabilities before deployment. Critical issues should be remediated before promoting images to production.

---

### Question

How would you support disaster recovery for ECR?

### Perfect Answer

I would configure cross-region replication so that images are automatically copied to another AWS Region, ensuring availability during regional outages.

---

### Question

Can Lambda use ECR images?

### Perfect Answer

Yes.

AWS Lambda supports container images stored in Amazon ECR, subject to supported image size and runtime requirements.

---

# 26. Hands-on Labs

## Lab 1

Create a private ECR repository.

---

## Lab 2

Authenticate Docker with ECR.

```bash
aws ecr get-login-password | docker login ...
```

---

## Lab 3

Tag a local image.

```bash
docker tag nginx:latest \
<account>.dkr.ecr.ap-south-1.amazonaws.com/nginx:v1
```

---

## Lab 4

Push the image.

```bash
docker push \
<account>.dkr.ecr.ap-south-1.amazonaws.com/nginx:v1
```

---

## Lab 5

Enable scan-on-push.

---

## Lab 6

Create a lifecycle policy to retain only the latest 10 images.

---

# 27. One-Page Revision

```
Docker Build

↓

Amazon ECR

↓

Image Repository

↓

IAM Authentication

↓

ECS / EKS / Lambda
```

Remember

- Private Repository
- Public Repository
- IAM
- Image Tags
- Lifecycle Policies
- Scan-on-Push
- Cross-Region Replication
- Cross-Account Access

---

# 28. Think Like a Production Engineer

Don't think:

> "ECR stores Docker images."

Think:

> "ECR is the secure image source of truth for production deployments."

Whenever someone says:

```
Deployment Failed

↓

Image Pull Error

↓

ECR
```

Your troubleshooting flow should be:

```
Repository Exists

↓

Image Tag

↓

IAM Role

↓

Authentication

↓

Network Connectivity

↓

Image Scan Status
```

---

# Key Takeaways

Amazon ECR is the central image registry for AWS container workloads. A production-ready implementation uses private repositories, immutable image tags, lifecycle policies, vulnerability scanning, IAM least privilege, and automated CI/CD integration.

# End of Chapter 06
