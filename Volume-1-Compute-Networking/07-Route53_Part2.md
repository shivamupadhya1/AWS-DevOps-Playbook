# Amazon Route 53

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 07
>
> Part 2 – Routing Policies & Production Architectures

---

# 15. Routing Policies

A Routing Policy tells Route 53 **how to answer a DNS query**.

Think of it like this:

```
User Request

↓

Route53

↓

Routing Policy

↓

Choose Destination
```

Different business requirements require different routing strategies.

---

# 16. Simple Routing

⭐⭐⭐

The default routing policy.

Example

```
www.company.com

↓

ALB
```

Every user receives the same DNS answer.

Use Case

- Small applications
- Single-region deployments
- Development environments

---

# 17. Weighted Routing

⭐⭐⭐⭐⭐

One of the most frequently asked interview topics.

Business Problem

You deployed

```
Version 1

and

Version 2
```

Question

How do you gradually move users to Version 2?

Weighted Routing.

Example

```
Version 1

Weight = 90
```

```
Version 2

Weight = 10
```

Approximately 90% of DNS responses point to Version 1 and 10% to Version 2.

Later

```
70 / 30

↓

50 / 50

↓

10 / 90

↓

0 / 100
```

---

### Production Use Cases

- Blue-Green deployments
- Canary releases
- A/B testing
- Gradual migrations

---

### Perfect Interview Answer

Weighted Routing distributes DNS responses according to configured weights, allowing gradual traffic shifts between multiple resources without changing application code.

---

# 18. Latency-Based Routing

Business Problem

Users

```
India

USA

Europe
```

Should everyone connect to

```
US-East-1?
```

No.

Route53 measures AWS Region latency and returns the resource with the lowest latency for that user.

Example

```
India

↓

ap-south-1
```

```
Germany

↓

eu-central-1
```

```
Virginia

↓

us-east-1
```

Benefits

- Lower response times
- Better user experience

---

# 19. Geolocation Routing

Do not confuse this with Latency Routing.

Geolocation uses the **user's geographic location**, not network latency.

Example

```
India

↓

Indian Website
```

```
France

↓

French Website
```

```
Japan

↓

Japanese Website
```

Common Uses

- Country-specific content
- Legal compliance
- Language selection

---

# 20. Geoproximity Routing

Available with Route 53 Traffic Flow.

Unlike Geolocation, Geoproximity routes traffic based on the **geographic location of AWS resources** and can shift traffic using **bias**.

Example

```
Region A

↓

Bias +20

↓

Receives More Users
```

Useful during migrations or regional capacity adjustments.

---

# 21. Failover Routing

⭐⭐⭐⭐⭐

Business Problem

Your primary application is deployed in

```
Mumbai
```

A disaster occurs.

How do users automatically switch to another Region?

Architecture

```
Primary

↓

Health Check

↓

Healthy?

↓

YES

↓

Primary

NO

↓

Secondary Region
```

This is called **Active-Passive Disaster Recovery**.

---

### Perfect Interview Answer

Failover Routing monitors the health of the primary endpoint. If it becomes unhealthy, Route 53 automatically returns the secondary endpoint until the primary recovers.

---

# 22. Multivalue Answer Routing

Suppose you have:

```
EC2-1

EC2-2

EC2-3
```

Route53 can return multiple healthy IP addresses in the DNS response.

Clients choose one of the returned addresses.

This provides basic load distribution and removes unhealthy endpoints from DNS responses.

---

# 23. Health Checks

Health Checks continuously monitor endpoints.

Example

```
GET /health
```

Expected

```
200 OK
```

If the endpoint fails, Route53 can stop returning that record depending on the routing policy.

Health Checks can monitor:

- HTTP
- HTTPS
- TCP

---

# 24. DNS TTL

TTL

```
Time To Live
```

Example

```
TTL = 300 Seconds
```

A DNS resolver may cache the response for about five minutes before asking Route53 again.

---

### Production Example

Blue-Green Deployment

Before migration

```
TTL

300

↓

30 Seconds
```

Reason

Faster propagation of DNS changes.

After migration

Increase TTL again to reduce DNS query volume.

---

# 25. Production Scenario 1

Problem

You want to migrate

```
Old ALB

↓

New ALB
```

Without downtime.

Solution

```
Weighted Routing

90

↓

10

↓

50

↓

50

↓

100
```

Observe metrics before increasing traffic.

---

# 26. Production Scenario 2

Problem

Mumbai Region unavailable.

Solution

```
Health Check

↓

Fail

↓

Failover Routing

↓

Singapore Region
```

Users are redirected automatically.

---

# 27. Production Scenario 3

Problem

Users in Europe experience high latency.

Solution

Use

```
Latency Routing
```

Deploy workloads in multiple Regions and allow Route53 to return the lowest-latency endpoint.

---

# 28. Active-Active vs Active-Passive

| Feature | Active-Active | Active-Passive |
|----------|---------------|----------------|
| Both Regions Serve Traffic | ✅ | ❌ |
| Disaster Recovery | Excellent | Good |
| Cost | Higher | Lower |
| Typical Routing | Latency / Weighted | Failover |

---

# 29. Best Practices

- Use health checks for critical endpoints.
- Lower TTL before planned DNS changes.
- Use Alias records for AWS services.
- Test failover regularly.
- Monitor Route53 health check status.

---

# 30. Common Mistakes

❌ Using a TTL of 24 hours before migration.

Changes take much longer to propagate.

---

❌ Confusing Geolocation with Latency Routing.

Geolocation = user's location.

Latency = fastest AWS Region.

---

❌ Never testing disaster recovery.

Failover should be validated periodically.

---

# 31. Interview Questions

---

## Question 1

Difference between Weighted and Latency Routing?

### Perfect Answer

Weighted Routing distributes DNS responses according to configured percentages.

Latency Routing returns the endpoint with the lowest network latency for the client.

---

## Question 2

Difference between Geolocation and Latency Routing?

### Perfect Answer

Geolocation makes decisions based on the user's geographic location.

Latency Routing makes decisions based on measured network latency to AWS Regions.

---

## Question 3

When would you use Failover Routing?

### Perfect Answer

Failover Routing is used for Active-Passive disaster recovery. Route53 monitors the primary endpoint using health checks and automatically returns the secondary endpoint if the primary becomes unavailable.

---

## Question 4

How do you perform a blue-green deployment using Route53?

### Perfect Answer

Lower the DNS TTL before the deployment, then use Weighted Routing to gradually shift traffic from the old environment to the new one. Monitor application metrics and increase the weight only after verifying stability.

---

## Question 5

What is TTL?

### Perfect Answer

TTL (Time To Live) specifies how long DNS resolvers may cache a DNS response. Lower TTL values allow faster DNS changes but can increase DNS query traffic.

---

# 32. Amazon Cross Questions

---

### Question

Can Route53 detect that an application is returning HTTP 500 errors?

### Perfect Answer

Yes, if a Route53 health check is configured to monitor an HTTP/HTTPS endpoint and that endpoint is designed to fail appropriately. In practice, many teams expose a dedicated health endpoint (for example, `/health`) that reflects application readiness.

---

### Question

Does lowering TTL immediately update every user's DNS cache?

### Perfect Answer

No.

TTL affects future caching behavior. Some recursive resolvers or clients may retain cached records until the previous TTL expires.

---

### Question

Can Weighted Routing guarantee exactly 10% of users?

### Perfect Answer

No.

Weighted Routing influences DNS responses, not individual HTTP requests. Because of DNS caching and client resolver behavior, the actual distribution is approximate.

---

# 33. One-Page Revision Sheet

```
Simple

↓

One Endpoint

Weighted

↓

Traffic %

Latency

↓

Fastest Region

Geolocation

↓

User Country

Geoproximity

↓

Resource Location + Bias

Failover

↓

Primary → Secondary

Multivalue

↓

Multiple Healthy IPs
```

---

# 34. Think Like a Production Engineer

Don't think

> "Route53 resolves DNS."

Think

> "Route53 is my global traffic manager."

Whenever someone says

```
Blue-Green Deployment
```

or

```
Disaster Recovery
```

Your immediate thought process should be:

```
TTL

↓

Routing Policy

↓

Health Checks

↓

Traffic Shift

↓

Monitoring

↓

Rollback (if required)
```

---

# Key Takeaways

Route53 is much more than a DNS service. It is a global traffic management platform that enables intelligent routing, disaster recovery, multi-region architectures, and controlled application deployments. Choosing the correct routing policy is a critical design decision that directly impacts availability, performance, and user experience.
