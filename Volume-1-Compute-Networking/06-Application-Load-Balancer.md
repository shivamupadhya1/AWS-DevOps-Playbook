# Application Load Balancer (ALB)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 06

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain why ALB exists.
- Understand Layer 7 load balancing.
- Explain listeners, listener rules, and target groups.
- Troubleshoot ALB issues in production.
- Design highly available architectures.
- Answer advanced interview questions confidently.

---

# 1. Business Problem

Imagine your application is running on one EC2 instance.

```
Internet

↓

EC2

↓

Application
```

Traffic:

```
500 Users
```

Everything works.

Now the application becomes popular.

```
50,000 Users
```

Problems:

- One server cannot handle all requests.
- If the EC2 fails, the application is unavailable.
- There is no automatic traffic distribution.

AWS introduced the **Application Load Balancer (ALB)** to solve these problems.

---

# 2. What is an Application Load Balancer?

An Application Load Balancer distributes incoming **HTTP and HTTPS (Layer 7)** traffic across multiple targets.

Supported targets include:

- EC2 Instances
- ECS Tasks
- EKS Pods (through AWS Load Balancer Controller)
- IP Addresses
- Lambda Functions

ALB understands HTTP requests and can make routing decisions based on:

- URL Path
- Host Header
- HTTP Method
- Query String
- Source IP (limited rule support)

---

# 3. Internal Architecture

```
Internet

↓

Route53

↓

ALB

↓

Listener (80 / 443)

↓

Listener Rules

↓

Target Group

↓

EC2 / ECS / EKS

↓

Application
```

---

# 4. Components of ALB

---

## Listener

A Listener checks incoming requests on a specific port.

Examples:

```
HTTP → Port 80

HTTPS → Port 443
```

Without a listener, ALB cannot accept traffic.

---

## Listener Rules

Rules determine where requests go.

Example:

```
/api/*

↓

API Server
```

```
/admin/*

↓

Admin Server
```

```
/images/*

↓

Image Service
```

---

## Target Group

A Target Group contains backend resources.

Example:

```
Target Group

↓

EC2-1

EC2-2

EC2-3
```

ALB sends traffic only to healthy targets.

---

## Health Checks

Every few seconds ALB calls a configured endpoint.

Example:

```
GET /health
```

Expected:

```
HTTP 200
```

If the endpoint fails repeatedly, the target is marked unhealthy.

---

# 5. Request Flow

```
User

↓

DNS (Route53)

↓

ALB

↓

Listener

↓

Listener Rule

↓

Target Group

↓

Healthy Target

↓

Application
```

---

# 6. Path-Based Routing

One ALB can route traffic to multiple services.

Example:

```
example.com/api

↓

API Service
```

```
example.com/admin

↓

Admin Service
```

```
example.com/payment

↓

Payment Service
```

---

# 7. Host-Based Routing

Example:

```
api.company.com

↓

API Servers
```

```
admin.company.com

↓

Admin Servers
```

```
shop.company.com

↓

Shopping Service
```

This allows multiple applications to share one ALB.

---

# 8. SSL Termination

Instead of configuring HTTPS on every EC2:

```
User

↓

HTTPS

↓

ALB

↓

Decrypt

↓

HTTP

↓

EC2
```

The SSL certificate is attached to the ALB using **AWS Certificate Manager (ACM)**.

Benefits:

- Centralized certificate management.
- Simpler backend configuration.
- Easier certificate renewal.

---

# 9. Production Architecture

```
Internet

↓

Route53

↓

ALB

↓

Target Group

↓

Auto Scaling Group

↓

EC2

↓

RDS
```

---

# 10. Health Check Deep Dive

Example configuration:

```
Path

/health

Port

8080

Protocol

HTTP

Healthy Threshold

5

Unhealthy Threshold

2

Timeout

5 sec

Interval

30 sec
```

If `/health` returns `200 OK`, traffic continues.

If it returns `500` or times out repeatedly, ALB removes the target.

---

# 11. Production Scenarios

## Scenario 1

Problem

Users receive:

```
503 Service Unavailable
```

### Possible Causes

- All targets unhealthy
- Wrong health check path
- Wrong health check port
- Application crashed
- Security Group blocking traffic

---

## Scenario 2

Problem

```
502 Bad Gateway
```

Possible Causes

- Backend application crashed
- Wrong target port
- Backend closed the connection
- Application timeout
- Reverse proxy misconfiguration

---

## Scenario 3

Problem

Only one EC2 receives traffic.

Possible Causes

- Other targets unhealthy
- Wrong target registration
- Sticky sessions enabled unexpectedly
- Incorrect listener rule

---

## Scenario 4

Problem

HTTPS works.

HTTP does not redirect.

Investigation

- Listener rule missing
- Redirect action not configured

---

# 12. Sticky Sessions

Normally

```
Request 1

↓

EC2-1

Request 2

↓

EC2-2
```

With sticky sessions:

```
User

↓

EC2-1

↓

All Future Requests

↓

EC2-1
```

Use only when required because it can lead to uneven traffic distribution.

---

# 13. Cross-Zone Load Balancing

Suppose

```
AZ-A

1 EC2
```

```
AZ-B

5 EC2
```

With Cross-Zone Load Balancing enabled:

Traffic is distributed across healthy targets in all enabled Availability Zones instead of being limited by the originating AZ.

This improves utilization and resilience.

---

# 14. Best Practices

- Use HTTPS.
- Store certificates in ACM.
- Enable health checks.
- Use multiple Availability Zones.
- Keep health check endpoints lightweight.
- Enable access logs if required.
- Use security groups with least privilege.

---

# 15. Common Mistakes

❌ Health check endpoint depends on the database.

If the database is down, every target becomes unhealthy.

Health endpoints should verify only essential application readiness unless deeper checks are intentionally required.

---

❌ Using `/` as a health check.

Home pages often perform expensive operations.

Create a dedicated `/health` endpoint.

---

❌ Opening backend security groups to the Internet.

Allow traffic only from the ALB security group.

---

# 16. Interview Questions

---

## Question 1

Why did AWS introduce ALB?

### Perfect Answer

ALB distributes Layer 7 HTTP/HTTPS traffic across multiple backend targets. It improves availability, supports intelligent routing based on request content, and enables applications to scale horizontally.

---

## Question 2

Difference between ALB and NLB?

### Perfect Answer

| ALB | NLB |
|------|------|
| Layer 7 | Layer 4 |
| HTTP/HTTPS | TCP/UDP/TLS |
| Path-based routing | No |
| Host-based routing | No |
| Web Applications | Low-latency network workloads |

---

## Question 3

Difference between ALB and Classic Load Balancer?

### Perfect Answer

Classic Load Balancer is the older generation and has more limited routing capabilities.

ALB supports:

- Path-based routing
- Host-based routing
- ECS integration
- EKS integration
- Lambda targets
- Modern HTTP features

---

## Question 4

Can one ALB route to multiple applications?

### Perfect Answer

Yes.

Using listener rules, one ALB can route requests to different target groups based on host names or URL paths.

---

## Question 5

Why are Target Groups important?

### Perfect Answer

Target Groups define where ALB sends traffic and continuously monitor the health of registered targets. Only healthy targets receive production traffic.

---

# 17. Amazon Cross Questions

### Question

Can one Target Group contain both ECS Tasks and EC2 instances?

### Perfect Answer

Generally, a target group uses one target type (Instance, IP, or Lambda). ECS services often register tasks using the **IP** target type (especially with Fargate), while EC2-based services commonly use the **Instance** target type.

---

### Question

Can an ALB exist without a Target Group?

### Perfect Answer

No.

Listener rules forward requests to target groups. Without a target group, the ALB cannot forward application traffic.

---

### Question

Does ALB terminate SSL?

### Perfect Answer

Yes.

A common architecture is SSL termination at the ALB using ACM certificates. The backend may use HTTP or HTTPS depending on security requirements.

---

# 18. Hands-on Lab

Objective:

Deploy an application behind an ALB.

Steps:

1. Launch two EC2 instances.
2. Install Nginx.
3. Create different index pages.
4. Create a Target Group.
5. Register both instances.
6. Create an ALB.
7. Configure Listener on port 80.
8. Forward traffic to the Target Group.
9. Refresh the browser repeatedly.
10. Stop one instance.
11. Observe health checks and traffic routing.
12. Add HTTPS using ACM (optional).

---

# 19. ALB vs NLB vs Gateway Load Balancer

| Feature | ALB | NLB | GWLB |
|---------|-----|-----|------|
| OSI Layer | 7 | 4 | 3 |
| HTTP/HTTPS | ✅ | ❌ | ❌ |
| TCP/UDP | ❌ | ✅ | ❌ |
| Path Routing | ✅ | ❌ | ❌ |
| Host Routing | ✅ | ❌ | ❌ |
| Security Appliances | ❌ | ❌ | ✅ |

---

# 20. One-Page Revision

```
User

↓

Route53

↓

ALB

↓

Listener

↓

Rule

↓

Target Group

↓

Healthy Target

↓

Application
```

Remember:

- Layer 7
- HTTP/HTTPS
- Listener
- Listener Rules
- Target Group
- Health Checks
- SSL Termination
- Path Routing
- Host Routing
- Cross-Zone Load Balancing

---

# 21. Think Like a Production Engineer

Don't think:

> "ALB forwards traffic."

Think:

> "ALB protects users from unhealthy applications."

When users report:

```
Website Down
```

Your troubleshooting flow should be:

```
DNS

↓

ALB

↓

Listener

↓

Listener Rules

↓

Target Group

↓

Health Checks

↓

EC2 / ECS / EKS

↓

Application Logs
```

---

# Key Takeaways

Application Load Balancer is the primary Layer 7 load balancer for modern AWS applications. It provides intelligent request routing, integrates with Auto Scaling, ECS, and EKS, performs continuous health checks, and improves both scalability and availability. For production systems, understanding listener rules, target groups, health checks, and troubleshooting flows is far more important than simply knowing how to create an ALB.
