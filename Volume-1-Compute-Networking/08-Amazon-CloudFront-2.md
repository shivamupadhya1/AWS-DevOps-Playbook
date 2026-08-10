# Amazon CloudFront

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 08
>
> Part 2 – Caching, Security & Production

---

# 20. Cache Behavior

⭐⭐⭐⭐⭐

Cache Behavior defines **how CloudFront handles incoming requests**.

Think of it as a rule book.

```
Incoming Request

↓

Cache Behavior

↓

Serve Cache

OR

Forward to Origin
```

Each behavior contains

- Path Pattern
- Origin
- Cache Policy
- Origin Request Policy
- Allowed HTTP Methods
- Viewer Protocol Policy

Example

```
/images/*

↓

S3

(Cache Enabled)
```

```
/api/*

↓

ALB

(Cache Disabled)
```

---

# 21. Cache Policy

Cache Policy decides

```
What gets cached?

How long?

Which headers?

Which cookies?

Which query strings?
```

Without a cache policy,

CloudFront doesn't know what content should remain in Edge Locations.

---

# 22. TTL (Time To Live)

⭐⭐⭐⭐⭐

TTL controls how long CloudFront keeps an object in cache.

There are three values:

### Minimum TTL

Smallest cache duration.

### Default TTL

Used when origin doesn't specify caching headers.

### Maximum TTL

Upper limit for cache duration.

Example

```
Minimum TTL

0 Seconds

Default TTL

3600 Seconds

Maximum TTL

86400 Seconds
```

---

### Production Example

Images

```
TTL

30 Days
```

API

```
TTL

0 Seconds
```

Reason

Images rarely change.

API responses change frequently.

---

# 23. Cache Invalidation

Business Problem

You deployed

```
logo.png
```

Users still see

Old Logo.

Reason

Edge Locations still have the old file.

Solution

Invalidate Cache.

Example

```
/logo.png
```

or

```
/*
```

CloudFront removes cached copies and fetches new content from the origin.

---

# 24. Origin Request Policy

Very common interview topic.

Question

Should CloudFront forward

- Headers?
- Cookies?
- Query Strings?

to the origin?

Origin Request Policy controls this.

Example

```
Authorization Header

↓

Forward

Language Header

↓

Forward

Everything Else

↓

Ignore
```

This reduces unnecessary origin requests.

---

# 25. Query String Caching

Suppose

```
product?id=10

product?id=20
```

Question

Should these be treated as the same page?

Usually

No.

CloudFront can cache different responses based on query parameters.

---

# 26. Cookie-Based Caching

Example

```
User A

↓

Premium User
```

```
User B

↓

Free User
```

Different cookies

↓

Different cached responses.

Useful for applications with personalized content.

---

# 27. Header-Based Caching

Example

```
Accept-Language

English
```

↓

English Page

```
Accept-Language

French
```

↓

French Page

CloudFront can cache different versions based on selected headers.

---

# 28. Compression

CloudFront supports

```
Gzip

+

Brotli
```

Benefits

- Smaller responses
- Faster downloads
- Lower bandwidth usage

Always enable compression for text-based content such as HTML, CSS, JavaScript, and JSON.

---

# 29. Signed URLs

⭐⭐⭐⭐⭐

Business Problem

Premium Video

Only paid users should access it.

CloudFront Signed URL

```
User

↓

Signed URL

↓

Valid?

↓

YES

↓

Download

NO

↓

403 Forbidden
```

Signed URLs include:

- Expiration time
- Signature
- Key Pair information

---

# 30. Signed Cookies

Instead of creating

100 Signed URLs

CloudFront can issue

One Signed Cookie

that grants access to multiple protected files.

Example

```
Movie

Trailer

Subtitle

Thumbnail
```

One cookie

↓

Access to all.

---

# 31. Origin Access Control (OAC)

⭐⭐⭐⭐⭐

Older Method

```
OAI

Origin Access Identity
```

Modern Method

```
OAC

Origin Access Control
```

OAC securely allows CloudFront to access private S3 buckets.

Architecture

```
User

↓

CloudFront

↓

OAC

↓

Private S3
```

Bucket remains private.

Users cannot bypass CloudFront.

---

# 32. CloudFront + WAF

Production Architecture

```
User

↓

CloudFront

↓

AWS WAF

↓

ALB

↓

Application
```

Benefits

- SQL Injection protection
- XSS protection
- IP blocking
- Rate limiting
- Geo restrictions

---

# 33. Production Scenario 1

Problem

Website updated.

Users still see old CSS.

Solution

```
Invalidate

/style.css
```

or use versioned filenames such as:

```
style-v2.css
```

---

# 34. Production Scenario 2

Problem

API responses are cached.

Users receive outdated information.

Solution

Configure

```
TTL

0 Seconds
```

or create a cache policy that bypasses caching for dynamic endpoints.

---

# 35. Production Scenario 3

Problem

Users access S3 bucket directly.

Security Risk.

Solution

Disable public bucket access.

Enable

```
CloudFront

+

OAC
```

Only CloudFront can read the bucket.

---

# 36. Production Scenario 4

Problem

European users receive English content.

Solution

Enable

Header-Based Caching

using

```
Accept-Language
```

---

# 37. Best Practices

✅ Enable Compression.

✅ Use OAC instead of OAI.

✅ Cache static assets aggressively.

✅ Do not cache sensitive API responses unless required.

✅ Use versioned static files.

✅ Protect origins using CloudFront.

---

# 38. Common Mistakes

❌ Invalidating

```
/*
```

after every deployment.

Large invalidations are slower and may incur additional cost.

Prefer versioned assets.

---

❌ Caching login responses.

Authentication responses are typically user-specific.

---

❌ Public S3 bucket with CloudFront.

Keep the bucket private.

---

# 39. Interview Questions

---

## Question 1

Difference between OAI and OAC?

### Perfect Answer

OAI (Origin Access Identity) is the older mechanism for allowing CloudFront to access private S3 content.

OAC (Origin Access Control) is the newer recommended approach with improved security and support for modern AWS features.

---

## Question 2

Difference between Signed URL and Signed Cookie?

### Perfect Answer

Signed URLs provide access to a single resource.

Signed Cookies allow access to multiple protected resources without generating a separate URL for each object.

---

## Question 3

What is Cache Invalidation?

### Perfect Answer

Cache Invalidation removes cached objects from Edge Locations so that CloudFront retrieves fresh content from the origin on subsequent requests.

---

## Question 4

How do you ensure users always receive the latest CSS after deployment?

### Perfect Answer

The preferred approach is to use versioned file names (for example, `style.v2.css`). If that's not possible, perform a targeted CloudFront cache invalidation.

---

## Question 5

Should APIs always be cached?

### Perfect Answer

No.

It depends on the API.

Static or infrequently changing responses may benefit from caching.

User-specific or frequently changing responses generally should not be cached.

---

# 40. Amazon Cross Questions

---

### Question

If CloudFront cache is empty, where does it retrieve data from?

### Perfect Answer

CloudFront forwards the request to the configured origin, retrieves the object, stores it according to the cache policy, and serves it to the user.

---

### Question

Can CloudFront cache POST requests?

### Perfect Answer

Typically, CloudFront caches **GET** and **HEAD** requests. POST requests are generally forwarded to the origin because they usually modify data or produce non-cacheable responses.

---

### Question

Should CloudFront cache login pages?

### Perfect Answer

Generally no.

Login pages and authentication responses are dynamic and often contain user-specific information.

---

# 41. One-Page Revision

```
CloudFront

↓

Cache Behavior

↓

Cache Policy

↓

TTL

↓

Edge Cache

↓

Origin
```

Remember

- Cache Behavior
- Cache Policy
- TTL
- Invalidation
- OAC
- Signed URL
- Signed Cookie
- Compression
- Query Strings
- Cookies
- Headers

---

# 42. Think Like a Production Engineer

Don't think

> "CloudFront caches files."

Think

> "CloudFront reduces origin load while securely delivering content worldwide."

Whenever someone says

```
Website Slow

↓

Old Content

↓

S3 Security

↓

Premium Videos
```

Immediately think

```
TTL

↓

Cache Policy

↓

OAC

↓

Signed URL

↓

Invalidation
```

---

# Key Takeaways

CloudFront caching is not just about speed. Proper cache policies, TTL configuration, origin request policies, secure origin access, and content protection are essential for building scalable, secure, and cost-efficient applications. Production engineers optimize cache behavior carefully to maximize performance without serving stale or incorrect data.

# End of Part 2
