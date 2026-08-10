# Amazon CloudFront

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 08
>
> Part 1 – Fundamentals & Architecture

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain why CloudFront exists.
- Understand CDN concepts.
- Explain Edge Locations.
- Understand Cache Behavior.
- Explain Origins.
- Understand the request flow.
- Answer CloudFront interview questions confidently.

---

# 1. Business Problem

Suppose your application is hosted in

```
Mumbai (ap-south-1)
```

Users access your application from:

```
India

USA

Germany

Australia
```

Question

Will users in Germany experience the same latency as users in Mumbai?

No.

The request must travel thousands of kilometers before reaching the application.

Problems:

- High latency
- Slow image loading
- Slow CSS/JavaScript delivery
- Poor user experience

AWS solves this using a CDN.

---

# 2. What is a CDN?

CDN stands for

```
Content Delivery Network
```

Instead of serving content only from the origin server,

AWS stores cached copies at locations around the world.

Nearest location serves the request.

```
User

↓

Nearest Edge Location

↓

Origin (Only if Needed)
```

---

# 3. What is CloudFront?

Amazon CloudFront is AWS's global Content Delivery Network.

It caches content closer to users to reduce latency, improve performance, and reduce load on origin servers.

CloudFront supports:

- Static Content
- Dynamic Content
- APIs
- Video Streaming
- Software Downloads

---

# 4. CloudFront Architecture

```
User

↓

Route53

↓

CloudFront

↓

Edge Location

↓

Origin

↓

ALB

↓

EC2 / ECS / EKS
```

OR

```
User

↓

CloudFront

↓

S3
```

---

# 5. Edge Locations

⭐⭐⭐⭐⭐

One of the most important interview topics.

Edge Locations are AWS locations distributed globally.

They cache content close to users.

Example

```
Origin

Mumbai
```

User

```
London
```

Instead of every request going to Mumbai,

CloudFront serves cached content from the London Edge Location.

---

# 6. Origin

Origin is where CloudFront fetches content from.

Supported origins:

- S3 Bucket
- Application Load Balancer
- EC2
- API Gateway
- Custom HTTP Server

CloudFront never creates content.

It only caches content from the origin.

---

# 7. Cache Flow

First request

```
User

↓

CloudFront

↓

Cache Miss

↓

Origin

↓

Response

↓

Store in Cache
```

Second request

```
User

↓

CloudFront

↓

Cache Hit

↓

Response
```

No request reaches the origin.

---

# 8. Cache Hit vs Cache Miss

Cache Hit

```
Content

Already Exists

↓

Serve Immediately
```

Cache Miss

```
Content Not Cached

↓

Fetch from Origin

↓

Store in Cache

↓

Return Response
```

---

# 9. Benefits of CloudFront

- Faster response times
- Lower latency
- Reduced origin load
- Better user experience
- Improved scalability
- Lower bandwidth consumption

---

# 10. CloudFront with S3

Very common architecture.

```
User

↓

CloudFront

↓

S3 Bucket
```

Perfect for:

- Images
- CSS
- JavaScript
- Videos
- Downloads

---

# 11. CloudFront with ALB

```
User

↓

CloudFront

↓

ALB

↓

Target Group

↓

EC2
```

This architecture is common for dynamic web applications.

---

# 12. Production Architecture

```
User

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

RDS
```

---

# 13. Production Scenario

Problem

Your website is hosted only in Mumbai.

Users in Europe report slow page loads.

Solution

Deploy CloudFront.

Static assets are served from nearby Edge Locations, significantly reducing latency.

---

# 14. Best Practices

- Put CloudFront in front of public web applications.
- Cache static assets aggressively.
- Use HTTPS.
- Integrate with AWS WAF.
- Use Origin Access Control (OAC) for S3 origins.
- Compress content (Gzip/Brotli).

---

# 15. Common Mistakes

❌ Making the S3 bucket public when using CloudFront.

Instead, use OAC to allow only CloudFront to access the bucket.

---

❌ Caching dynamic API responses for too long.

Dynamic endpoints often require lower TTLs or no caching.

---

❌ Forgetting cache invalidation after deploying new static files.

Users may continue receiving old content.

---

# 16. Interview Questions

---

## Question 1

What is CloudFront?

### Perfect Answer

Amazon CloudFront is AWS's global CDN that caches content at Edge Locations to reduce latency, improve performance, and decrease load on origin servers.

---

## Question 2

What is an Edge Location?

### Perfect Answer

An Edge Location is a globally distributed AWS site where CloudFront caches content closer to end users, enabling faster content delivery.

---

## Question 3

What is an Origin?

### Perfect Answer

An Origin is the backend source from which CloudFront retrieves content. Common origins include S3 buckets, ALBs, EC2 instances, and API Gateway.

---

## Question 4

Difference between Cache Hit and Cache Miss?

### Perfect Answer

A Cache Hit occurs when CloudFront serves content directly from an Edge Location.

A Cache Miss occurs when CloudFront retrieves content from the origin, returns it to the client, and stores it in the cache for future requests.

---

## Question 5

Can CloudFront work with ALB?

### Perfect Answer

Yes.

CloudFront commonly uses an ALB as its origin for dynamic web applications, providing caching for eligible content and global request acceleration.

---

# 17. Amazon Cross Questions

### Question

Does CloudFront replace an ALB?

### Perfect Answer

No.

CloudFront is a CDN and caching layer.

ALB is a Layer 7 load balancer that distributes requests to backend targets.

They complement each other.

---

### Question

Can CloudFront cache API responses?

### Perfect Answer

Yes.

However, caching behavior should be configured carefully because many APIs return user-specific or frequently changing data.

---

### Question

If an Edge Location does not have the requested object, what happens?

### Perfect Answer

CloudFront forwards the request to the configured origin, retrieves the object, stores it according to cache settings, and returns it to the client.

---

# 18. One-Page Revision

```
User

↓

Route53

↓

CloudFront

↓

Edge Location

↓

Origin

↓

ALB / S3

↓

Application
```

Remember:

- CDN
- Edge Location
- Origin
- Cache Hit
- Cache Miss
- Static Content
- Dynamic Content
- OAC
- WAF Integration

---

# 19. Think Like a Production Engineer

Don't think:

> "CloudFront caches files."

Think:

> "CloudFront reduces latency, protects origins, and improves global scalability."

Whenever someone says:

```
Website Slow Worldwide
```

Your thought process should be:

```
CloudFront

↓

Edge Cache

↓

Origin

↓

Cache Policy

↓

TTL

↓

Performance
```

---

# Key Takeaways

CloudFront is the entry point for many modern AWS web applications. It improves user experience through global caching, reduces load on backend infrastructure, integrates with WAF for security, and works seamlessly with S3, ALB, ECS, and EKS.

# End of Part 1
