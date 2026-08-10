# Docker Security

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 05

---

# Chapter Objective

After completing this chapter, you should be able to:

- Build secure Docker images
- Reduce Docker image vulnerabilities
- Understand Linux Capabilities
- Explain Root vs Non-Root containers
- Understand Seccomp, AppArmor and SELinux
- Manage Docker Secrets securely
- Scan container images
- Secure production Docker deployments
- Answer Docker Security interview questions

---

# 1. Why Docker Security?

Suppose an attacker compromises your application.

If the container is running as

```
root
```

the attacker now has root privileges **inside the container**.

If combined with another vulnerability, this may allow access to the host.

Production systems must minimize this risk.

---

# 2. Docker Security Layers

Think of Docker security in multiple layers.

```
Docker Image

↓

Container Runtime

↓

Linux Kernel

↓

Host OS

↓

Cloud Infrastructure
```

Security should be applied at every layer.

---

# 3. Root vs Non-Root Container

⭐⭐⭐⭐⭐

Default

```
Container

↓

root
```

Example Dockerfile

```dockerfile
FROM nginx
```

Container starts as root.

Better approach

```dockerfile
FROM nginx

RUN useradd appuser

USER appuser
```

Now the application runs with limited privileges.

---

# 4. Why Avoid Root?

Problems

- File system modifications
- Package installation
- Process control
- Increased attack surface
- Privilege escalation risk

Best Practice

Run applications using a dedicated non-root user.

---

# 5. USER Instruction

Example

```dockerfile
FROM eclipse-temurin:21-jre

RUN useradd app

USER app

WORKDIR /app
```

Always verify the application works correctly with reduced privileges.

---

# 6. Image Vulnerabilities

Images contain

- Operating System packages
- Libraries
- Runtime
- Dependencies

Each may contain CVEs.

Example

```
Ubuntu

↓

OpenSSL

↓

Critical CVE
```

Even if your application is secure, the base image may not be.

---

# 7. Image Scanning

⭐⭐⭐⭐⭐

Never deploy images without scanning.

Popular tools

- Trivy
- Docker Scout
- Amazon ECR Image Scanning
- Snyk
- Grype
- Prisma Cloud

Example

```bash
trivy image myapp:1.0
```

The scan reports

- Critical
- High
- Medium
- Low vulnerabilities

---

# 8. Multi-Stage Builds

⭐⭐⭐⭐⭐

Instead of shipping build tools,

only ship the application.

Example

```dockerfile
FROM maven:3.9 AS build

COPY . .

RUN mvn clean package

FROM eclipse-temurin:21-jre

COPY --from=build target/app.jar app.jar

CMD ["java","-jar","app.jar"]
```

Benefits

- Smaller image
- Fewer packages
- Lower attack surface
- Faster deployment

---

# 9. Distroless Images

Google Distroless images contain

- Application
- Runtime

Only.

No

- Shell
- apt
- yum
- bash

Advantages

- Very small
- Highly secure
- Reduced attack surface

Disadvantage

Debugging is harder because common tools are absent.

---

# 10. Docker Secrets

Never do this

```dockerfile
ENV DB_PASSWORD=password123
```

or

```yaml
environment:
  PASSWORD=password
```

Instead use

- Docker Secrets
- AWS Secrets Manager
- HashiCorp Vault
- Kubernetes Secrets

---

# 11. .dockerignore

Just like `.gitignore`.

Example

```
.git

target

node_modules

*.log

.env
```

Benefits

- Faster builds
- Smaller context
- Prevents sensitive files from entering images

---

# 12. Read-Only Filesystem

Example

```bash
docker run \
--read-only \
nginx
```

The container cannot modify its filesystem.

Useful for stateless applications.

---

# 13. Drop Linux Capabilities

⭐⭐⭐⭐⭐

Linux divides root privileges into capabilities.

Examples

```
CAP_NET_ADMIN

CAP_SYS_ADMIN

CAP_SYS_PTRACE
```

Containers should receive only the capabilities they need.

Example

```bash
docker run \
--cap-drop ALL \
--cap-add NET_BIND_SERVICE \
nginx
```

Principle

**Least Privilege**

---

# 14. Seccomp

Seccomp restricts Linux system calls.

Example

```
Application

↓

Allowed System Calls

↓

Kernel
```

Dangerous calls can be blocked.

Docker applies a default Seccomp profile.

---

# 15. AppArmor

AppArmor restricts what processes can access.

Examples

- Files
- Directories
- Capabilities
- Networking

Primarily available on Ubuntu-based systems.

---

# 16. SELinux

Primarily used on Red Hat, CentOS and Fedora.

Adds mandatory access control.

Useful for preventing unauthorized file access between containers and the host.

---

# 17. Root Filesystem Protection

Run

```bash
docker run \
--read-only \
--tmpfs /tmp \
nginx
```

The application can use `/tmp` while the rest of the filesystem remains read-only.

---

# 18. Limit Resources

Avoid unlimited CPU and memory.

Example

```bash
docker run \
--memory=512m \
--cpus=1 \
nginx
```

Protects the host from resource exhaustion.

---

# 19. Sign Images

Docker Content Trust can verify image integrity.

This helps ensure images haven't been tampered with.

---

# 20. Image Updates

Don't keep outdated images.

Example

```
Ubuntu 18

↓

Unsupported
```

Regularly rebuild images with updated base images and dependencies.

---

# 21. Production Architecture

```
Developer

↓

Docker Build

↓

Image Scan

↓

CI/CD Approval

↓

Container Registry

↓

Production
```

Images should be scanned before reaching production.

---

# 22. Production Scenario 1

Problem

Image contains 200 vulnerabilities.

Action

- Update base image
- Upgrade packages
- Remove unnecessary software
- Rebuild image
- Scan again

---

# 23. Production Scenario 2

Problem

Container compromised.

Check

- Running user
- Mounted volumes
- Linux capabilities
- Secrets exposure
- Application logs

---

# 24. Production Scenario 3

Problem

Developer stored AWS credentials inside Dockerfile.

Risk

Anyone with the image can extract them.

Solution

Use AWS Secrets Manager or IAM Roles.

---

# 25. Production Scenario 4

Problem

Image size is 1.5 GB.

Solution

- Multi-stage builds
- Remove build tools
- Use slim or distroless images
- Remove unused packages

---

# 26. Best Practices

- Run as a non-root user.
- Use official trusted base images.
- Scan every image.
- Use multi-stage builds.
- Keep images updated.
- Store secrets externally.
- Drop unnecessary Linux capabilities.
- Use read-only filesystems where possible.
- Limit CPU and memory.

---

# 27. Common Mistakes

❌ Running every container as root.

---

❌ Embedding passwords in Dockerfiles.

---

❌ Ignoring vulnerability scan results.

---

❌ Using outdated base images.

---

❌ Shipping build tools in production images.

---

# 28. Interview Questions

## Question 1

Why should Docker containers not run as root?

### Perfect Answer

Running as root increases the attack surface. If an attacker compromises the application, they gain root privileges inside the container. Running as a dedicated non-root user follows the principle of least privilege and reduces risk.

---

## Question 2

What is a Distroless Image?

### Perfect Answer

A Distroless image contains only the application and its runtime, without shells or package managers. This reduces image size and minimizes the attack surface.

---

## Question 3

How do you reduce Docker image vulnerabilities?

### Perfect Answer

I use trusted base images, keep them updated, perform image scanning, remove unnecessary packages, use multi-stage builds, and rebuild images regularly.

---

## Question 4

How should secrets be managed?

### Perfect Answer

Secrets should not be stored in Dockerfiles or source code. In production, I use AWS Secrets Manager, Docker Secrets, or HashiCorp Vault depending on the platform.

---

## Question 5

What are Linux Capabilities?

### Perfect Answer

Linux Capabilities divide root privileges into smaller permissions. Docker containers should receive only the capabilities required by the application, following the principle of least privilege.

---

# 29. Amazon Cross Questions

### Question

Does running a non-root user make a container completely secure?

### Perfect Answer

No.

It significantly reduces risk but should be combined with image scanning, capability restrictions, secure secrets management, resource limits, and kernel-level protections.

---

### Question

Would you use `latest` images in production?

### Perfect Answer

No.

I use explicit version tags to ensure repeatable deployments and controlled upgrades.

---

### Question

How do you secure Docker images in a CI/CD pipeline?

### Perfect Answer

Build the image, scan it for vulnerabilities, fail the pipeline if critical issues are detected, sign the image if required, push it to a trusted registry, and deploy only approved images.

---

# 30. Hands-on Labs

## Lab 1

Run a container as a non-root user.

---

## Lab 2

Scan an image using Trivy.

```bash
trivy image nginx:latest
```

---

## Lab 3

Create a multi-stage Dockerfile.

---

## Lab 4

Add a `.dockerignore` file.

---

## Lab 5

Run a container with limited memory.

```bash
docker run \
--memory=256m \
nginx
```

---

## Lab 6

Run a container with dropped capabilities.

```bash
docker run \
--cap-drop ALL \
--cap-add NET_BIND_SERVICE \
nginx
```

---

# 31. One-Page Revision

```
Dockerfile

↓

Non-Root User

↓

Image Scan

↓

Multi-Stage Build

↓

Secrets Manager

↓

Registry

↓

Production
```

Remember

- Non-root user
- Multi-stage builds
- Distroless images
- Trivy
- Docker Scout
- Capabilities
- Seccomp
- AppArmor
- SELinux
- Secrets
- `.dockerignore`

---

# 32. Think Like a Production Engineer

Don't think:

> "The container works."

Think:

> "Is it secure enough for production?"

Whenever someone says:

```
Security Review

↓

Container Scan

↓

Critical CVEs

↓

Root User

↓

Secrets Exposure
```

Your checklist should be:

```
Image Scan

↓

Base Image

↓

Running User

↓

Capabilities

↓

Secrets

↓

Filesystem

↓

Resource Limits
```

---

# Final Key Takeaways

Container security is built in layers. A secure production image should run as a non-root user, use minimal trusted base images, be scanned for vulnerabilities, avoid embedded secrets, limit Linux capabilities, and follow the principle of least privilege. Security is not a single feature—it is a continuous process integrated into the entire container lifecycle.

# End of Chapter 05
