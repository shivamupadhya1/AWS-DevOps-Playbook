# Amazon CloudFront

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 08
>
> Part 3 – Production, Monitoring & Troubleshooting

---

# 43. Multiple Origins

One CloudFront Distribution can have multiple origins.

Example

```
CloudFront

↓

Origin 1

S3

↓

Static Files

-----------------------

Origin 2

ALB

↓

Application

-----------------------

Origin 3

API Gateway

↓

REST API
```

CloudFront decides the origin using Cache Behaviors.

Example

```
/images/*

↓

S3

/api/*

↓

API Gateway

/*

↓

ALB
```

---

# 44. Origin Groups (Origin Failover)

⭐⭐⭐⭐⭐

Business Problem

Your application is hosted in

```
Primary ALB
```

Suppose

Primary ALB becomes unavailable.

Should users receive

```
503 Service Unavailable?
```

No.

CloudFront supports

Origin Groups.

Architecture

```
CloudFront

↓

Primary Origin

↓

Healthy?

↓

YES

↓

Serve User

NO

↓

Secondary Origin

↓

Serve User
```

This provides automatic origin failover.

---

# 45. Lambda@Edge

⭐⭐⭐⭐⭐

Lambda@Edge allows you to execute Lambda functions at AWS Edge Locations.

Example Use Cases

- Authentication
- URL Rewrite
- Header Modification
- Redirects
- Security
- A/B Testing

Flow

```
User

↓

Edge Location

↓

Lambda@Edge

↓

Origin
```

Example

```
Old URL

↓

Lambda

↓

New URL
```

without changing the application.

---

# 46. CloudFront Functions

CloudFront Functions are lightweight alternatives to Lambda@Edge.

Use Cases

- HTTP redirects
- Header manipulation
- Authentication checks
- URL normalization

Difference

| CloudFront Functions | Lambda@Edge |
|----------------------|-------------|
| Lightweight | Full Lambda Runtime |
| Very Low Latency | Higher |
| Less Expensive | More Expensive |
| Simple Logic | Complex Logic |

---

# 47. Geo Restriction

Business Problem

Application should only work in

```
India

Singapore
```

CloudFront can

Allow

or

Block

countries.

Example

```
India

Allow

USA

Block

China

Block
```

Very common for licensing restrictions.

---

# 48. CloudWatch Metrics

Important Metrics

```
Requests

BytesDownloaded

BytesUploaded

4xxErrorRate

5xxErrorRate

CacheHitRate

OriginLatency
```

Interview Tip

A low Cache Hit Rate usually means more requests are reaching the origin, increasing latency and cost.

---

# 49. CloudFront Logs

CloudFront supports

Standard Logs

↓

S3

Real-Time Logs

↓

Kinesis Data Streams

Useful Information

- Client IP
- URI
- Status Code
- Cache Hit/Miss
- User Agent
- Request Time

---

# 50. CloudFront Monitoring

Production Dashboard

```
CloudWatch

↓

Cache Hit Ratio

↓

Origin Latency

↓

5xx Errors

↓

Traffic

↓

Bandwidth
```

---

# 51. Production Scenario 1

Problem

Users receive

```
403 Forbidden
```

Possible Causes

- OAC/OAI misconfiguration
- Private S3 bucket permissions
- Missing signed URL/cookie
- AWS WAF blocking the request
- Geo restriction
- Incorrect bucket policy

---

# 52. Production Scenario 2

Problem

```
404 Not Found
```

Possible Causes

- File doesn't exist
- Wrong origin path
- Wrong cache behavior
- Incorrect deployment
- Wrong S3 object key

---

# 53. Production Scenario 3

Problem

```
502 Bad Gateway
```

Possible Causes

- ALB not reachable
- Origin SSL mismatch
- Backend application failure
- Wrong origin configuration
- DNS resolution issue for custom origin

---

# 54. Production Scenario 4

Problem

```
503 Service Unavailable
```

Possible Causes

- All origin servers unhealthy
- ALB target group unhealthy
- Origin overloaded
- Origin unavailable
- Rate limiting upstream

---

# 55. Production Scenario 5

Problem

```
504 Gateway Timeout
```

Possible Causes

- Origin timeout
- Database latency
- Long-running API
- Network issues
- Backend thread pool exhausted

---

# 56. Troubleshooting Flow

Website Slow

```
↓

Cache Hit Rate

↓

Low?

↓

Origin Latency

↓

High?

↓

ALB

↓

EC2

↓

Application

↓

Database
```

Always troubleshoot layer by layer.

---

# 57. AWS CLI

Create Invalidation

```bash
aws cloudfront create-invalidation \
--distribution-id E123456 \
--paths "/*"
```

Get Distribution

```bash
aws cloudfront get-distribution \
--id E123456
```

List Distributions

```bash
aws cloudfront list-distributions
```

---

# 58. Terraform Example

```hcl
resource "aws_cloudfront_distribution" "web" {

  enabled = true

  origin {

    domain_name = aws_lb.app.dns_name

    origin_id = "alb-origin"
  }

  default_cache_behavior {

    target_origin_id = "alb-origin"

    viewer_protocol_policy = "redirect-to-https"

    allowed_methods = [
      "GET",
      "HEAD"
    ]
  }

  viewer_certificate {

    cloudfront_default_certificate = true
  }
}
```

---

# 59. Hands-on Lab

Objective

Deploy CloudFront in front of an ALB.

Lab Steps

1. Create an ALB.
2. Deploy a sample application.
3. Create a CloudFront Distribution.
4. Set the ALB as the origin.
5. Enable compression.
6. Test Cache Hit and Cache Miss.
7. Perform cache invalidation.
8. Configure OAC for S3 (optional second origin).
9. Observe CloudWatch metrics.
10. Trigger a deployment and verify updated content.

---

# 60. Production Architecture

```
Users

↓

Route53

↓

CloudFront

↓

AWS WAF

↓

ALB

↓

Auto Scaling Group

↓

EC2

↓

Application

↓

RDS
```

This is one of the most common AWS production architectures.

---

# 61. Amazon Interview Questions

---

## Question 1

Why should CloudFront be placed before an ALB?

### Perfect Answer

CloudFront caches eligible content at Edge Locations, reduces requests reaching the ALB, lowers origin latency, and provides additional security features such as AWS WAF integration and geographic restrictions.

---

## Question 2

When would you use Lambda@Edge instead of CloudFront Functions?

### Perfect Answer

CloudFront Functions are ideal for lightweight request and response manipulation.

Lambda@Edge should be used when more advanced processing is required, such as authentication, complex request rewriting, or origin request customization.

---

## Question 3

Your Cache Hit Ratio is only 10%.

What does that indicate?

### Perfect Answer

Most requests are reaching the origin instead of being served from Edge Locations. I would review cache policies, TTL values, cache keys (headers, cookies, query strings), and determine whether dynamic content is preventing effective caching.

---

## Question 4

Users still receive old CSS after deployment.

What would you do?

### Perfect Answer

The preferred solution is versioned static assets (for example, `style.v2.css`). If necessary, create a targeted CloudFront invalidation for the affected files.

---

## Question 5

How do you secure a private S3 bucket behind CloudFront?

### Perfect Answer

Keep the S3 bucket private and use Origin Access Control (OAC) so only CloudFront can retrieve objects from the bucket.

---

## Question 6

What is the difference between Origin Failover and Route53 Failover?

### Perfect Answer

CloudFront Origin Failover switches between origins within a single distribution when the primary origin becomes unavailable.

Route53 Failover switches DNS responses between different endpoints or Regions based on health checks.

---

# 62. Interview Traps

### Trap 1

Interviewer

> Does CloudFront cache POST requests?

Correct Answer

Generally no.

CloudFront primarily caches GET and HEAD responses.

---

### Trap 2

Interviewer

> Should APIs always be cached?

Correct Answer

No.

Only APIs that return cacheable responses should be cached. Authentication and user-specific APIs are typically not cached.

---

### Trap 3

Interviewer

> Can users bypass CloudFront and access S3 directly?

Correct Answer

Not if OAC is correctly configured and the bucket is private.

---

# 63. CloudFront vs ALB vs Route53

| Feature | Route53 | CloudFront | ALB |
|----------|----------|------------|-----|
| DNS Resolution | ✅ | ❌ | ❌ |
| CDN | ❌ | ✅ | ❌ |
| Layer 7 Routing | ❌ | Limited (cache behaviors) | ✅ |
| Load Balancing | ❌ | ❌ | ✅ |
| Caching | ❌ | ✅ | ❌ |
| SSL Termination | ❌ | ✅ | ✅ |

---

# 64. One-Page Revision

```
User

↓

Route53

↓

CloudFront

↓

Edge Location

↓

WAF

↓

ALB

↓

Target Group

↓

EC2 / ECS / EKS

↓

Application
```

Remember

- Edge Locations
- Origins
- Cache Policies
- TTL
- OAC
- Signed URLs
- Signed Cookies
- Origin Groups
- Lambda@Edge
- CloudFront Functions
- Compression
- Geo Restriction

---

# 65. Think Like a Production Engineer

Don't think

> "CloudFront makes websites faster."

Think

> "CloudFront is the global entry point that improves performance, reduces origin load, enhances security, and increases availability."

Whenever someone says

```
Website Slow

↓

Global Users

↓

High Origin Load

↓

Large Static Files
```

Your immediate troubleshooting flow should be

```
Cache Hit Ratio

↓

TTL

↓

Origin Latency

↓

Cache Policy

↓

Origin Health

↓

Application
```

---

# Final Key Takeaways

CloudFront is much more than a CDN. It is a global edge platform that accelerates content delivery, secures applications, protects origins, integrates with AWS WAF, supports multi-origin architectures, and enables advanced request processing using Lambda@Edge and CloudFront Functions.

A production engineer should understand not only how to create a CloudFront distribution, but also how to optimize caching, secure origins, troubleshoot edge-to-origin failures, and design resilient global architectures.

# End of Chapter 08
