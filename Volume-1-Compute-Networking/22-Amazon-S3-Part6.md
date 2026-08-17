# Amazon S3

> AWS DevOps Playbook
>
> Volume 1 – Storage
>
> Chapter 22
>
> Amazon S3 – Part 6 (Advanced Features)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Static Website Hosting
- Learn Pre-Signed URLs
- Understand Multipart Upload
- Learn Transfer Acceleration
- Understand Event Notifications
- Learn S3 Object Lock
- Understand Access Points
- Learn Multi-Region Access Points
- Learn Performance Optimization
- Answer interview questions

---

# 143. Static Website Hosting

Amazon S3 can host static websites.

A static website contains only:

- HTML
- CSS
- JavaScript
- Images
- Fonts

There is no backend application running.

Example:

```
Browser

↓

Amazon S3

↓

index.html

↓

style.css

↓

logo.png
```

Examples:

- Company Portfolio
- Documentation
- Landing Page
- Resume Website

---

# 144. Static Website Architecture

```
User

↓

Internet

↓

Amazon S3 Bucket

↓

index.html

↓

CSS

↓

JavaScript

↓

Images
```

You enable:

```
Static Website Hosting
```

on the bucket.

---

# 145. Static Website + CloudFront

Production architecture:

```
Users

↓

CloudFront

↓

Amazon S3

↓

Website Files
```

Benefits:

- HTTPS
- Global CDN
- Better Performance
- DDoS Protection
- Lower Latency

This is the recommended production design.

---

# 146. Pre-Signed URL

Sometimes you want to share a private file temporarily.

Instead of making the bucket public:

```
Private Bucket

↓

Generate URL

↓

Valid For

1 Hour

↓

Share URL
```

The URL expires automatically.

---

# 147. Pre-Signed URL Example

Suppose:

```
invoice.pdf
```

Customer downloads:

```
https://...

↓

Valid

15 Minutes
```

After expiration:

```
Access Denied
```

Common Uses:

- Invoice downloads
- Medical reports
- Secure document sharing
- Temporary uploads

---

# 148. Multipart Upload

Maximum object size:

```
5 TB
```

Objects larger than **5 GB** should use Multipart Upload.

Instead of uploading:

```
20 GB

↓

Single Upload
```

Split into parts:

```
Part 1

Part 2

Part 3

...

Part N
```

AWS combines them after upload.

---

# 149. Multipart Upload Benefits

✔ Faster uploads

✔ Parallel upload

✔ Retry only failed parts

✔ Better reliability

✔ Better network utilization

---

# 150. Multipart Upload Example

```
100 GB File

↓

25 Parts

↓

Parallel Upload

↓

Combine

↓

One Object
```

If Part 17 fails:

Only Part 17 is uploaded again.

---

# 151. Byte-Range Fetch

Instead of downloading:

```
100 GB
```

You download:

```
Bytes

1000–5000
```

Useful for:

- Video Streaming
- Large Files
- Resume Downloads

---

# 152. Transfer Acceleration

Normally:

```
India

↓

Internet

↓

S3 Mumbai
```

If the client is far away:

```
Brazil

↓

Internet

↓

Mumbai
```

Higher latency.

Transfer Acceleration uses CloudFront Edge Locations.

---

# 153. Transfer Acceleration Architecture

```
Client

↓

Nearest CloudFront Edge

↓

AWS Backbone

↓

S3 Bucket
```

Benefits:

- Faster uploads
- Faster downloads
- Global optimization

Best suited for users distributed around the world.

---

# 154. Event Notifications

S3 can automatically trigger events when objects change.

Supported events include:

- Object Created
- Object Deleted
- Restore Completed
- Replication Events (selected cases)

Example:

```
Upload Image

↓

S3

↓

Event

↓

Lambda
```

---

# 155. Event Destinations

Amazon S3 supports sending events to:

```
Amazon SNS

Amazon SQS

AWS Lambda

Amazon EventBridge
```

This enables event-driven architectures.

---

# 156. Production Example

User uploads:

```
profile.jpg
```

Flow:

```
Upload

↓

Amazon S3

↓

Event Notification

↓

Lambda

↓

Resize Image

↓

Store Thumbnail
```

Very common interview scenario.

---

# 157. Object Lock

Object Lock prevents objects from being deleted or overwritten for a defined retention period.

Useful for:

- Banking
- Healthcare
- Compliance
- Legal Records

---

# 158. Retention Modes

### Governance Mode

```
Delete

↓

Denied

↓

Unless Authorized
```

Privileged users with appropriate permissions can bypass governance retention when necessary.

---

### Compliance Mode

```
Delete

↓

Denied

↓

Nobody Can Delete

↓

Until Retention Ends
```

Even administrators cannot bypass Compliance Mode before the retention period expires.

---

# 159. Legal Hold

Legal Hold differs from retention.

```
Object

↓

Legal Hold

↓

Cannot Delete

↓

Until Hold Removed
```

No expiration date is required.

Useful during:

- Court Cases
- Investigations
- Audits

---

# 160. Access Points

Suppose:

One bucket

```
company-data
```

Departments:

- HR
- Finance
- DevOps

Instead of one complicated bucket policy:

```
Bucket

↓

Access Point

↓

HR
```

```
Bucket

↓

Access Point

↓

Finance
```

```
Bucket

↓

Access Point

↓

DevOps
```

Each Access Point has its own permissions.

---

# 161. Benefits of Access Points

✔ Simpler policies

✔ Easier management

✔ Department isolation

✔ Large enterprise support

✔ Fine-grained access

---

# 162. Multi-Region Access Points

Suppose:

```
Mumbai Bucket

Singapore Bucket

London Bucket
```

Instead of choosing one manually:

```
Application

↓

Multi-Region Access Point

↓

Nearest Healthy Bucket
```

Benefits:

- Automatic routing
- Lower latency
- Regional failover
- Global applications

---

# 163. Performance Optimization

Amazon S3 automatically scales.

However, best practices include:

✔ Use Multipart Upload for large files.

✔ Use Byte-Range Fetch for large downloads.

✔ Use Transfer Acceleration for global users.

✔ Distribute object keys across prefixes for workloads that generate extremely high request rates.

✔ Use CloudFront for frequently accessed content.

---

# 164. Production Architecture

```
Users

↓

CloudFront

↓

Amazon S3

↓

Event Notification

↓

Lambda

↓

Image Processing

↓

Thumbnail Bucket
```

This is a common serverless image processing architecture.

---

# 165. Best Practices

✔ Keep buckets private.

✔ Use CloudFront instead of exposing S3 directly for websites.

✔ Use Pre-Signed URLs for temporary sharing.

✔ Use Multipart Upload for large files.

✔ Enable Object Lock where compliance requires immutable storage.

✔ Use Access Points in large organizations.

✔ Enable Transfer Acceleration for global workloads.

---

# 166. Common Mistakes

❌ Public buckets for confidential files.

❌ Uploading 100 GB files without Multipart Upload.

❌ Sharing permanent object URLs instead of Pre-Signed URLs.

❌ Not using CloudFront.

❌ No Event Notifications for automation.

❌ Overly complex bucket policies instead of Access Points.

---

# 167. Production Scenario 1

Problem:

Application uploads 50 GB backups.

Solution:

```
Multipart Upload
```

---

# 168. Production Scenario 2

Problem:

Customer downloads invoices.

Invoices must expire after 30 minutes.

Solution:

```
Pre-Signed URL
```

---

# 169. Production Scenario 3

Problem:

Users upload profile images.

Application must generate thumbnails.

Solution:

```
S3

↓

Lambda

↓

Thumbnail
```

---

# 170. Production Scenario 4

Problem:

Legal department requires files to remain undeletable for 7 years.

Solution:

```
Object Lock

↓

Compliance Mode
```

---

# 171. Production Scenario 5

Problem:

Global users experience slow uploads.

Solution:

```
Transfer Acceleration
```

---

# 172. Interview Questions

## Question 39

What is Static Website Hosting?

### Answer

A feature that allows an S3 bucket to host static website content such as HTML, CSS, JavaScript, and images.

---

## Question 40

What is a Pre-Signed URL?

### Answer

A temporary URL that grants time-limited access to a private S3 object.

---

## Question 41

When should Multipart Upload be used?

### Answer

Multipart Upload is recommended for objects larger than **5 GB** and is required for uploads larger than 5 GB because a single PUT request cannot exceed 5 GB.

---

## Question 42

What is Transfer Acceleration?

### Answer

A feature that speeds up uploads and downloads by routing traffic through the AWS global edge network before transferring it over the AWS backbone.

---

## Question 43

What services can receive S3 Event Notifications?

### Answer

- Amazon SNS
- Amazon SQS
- AWS Lambda
- Amazon EventBridge

---

## Question 44

What is Object Lock?

### Answer

Object Lock prevents objects from being deleted or overwritten for a configured retention period, supporting WORM (Write Once Read Many) requirements.

---

## Question 45

What is the difference between Governance Mode and Compliance Mode?

### Answer

Governance Mode allows authorized users with special permissions to bypass retention in certain cases, while Compliance Mode cannot be bypassed by any user until the retention period expires.

---

## Question 46

Why use Access Points?

### Answer

Access Points simplify permission management by providing separate access policies for different applications, teams, or departments using the same bucket.

---

## Question 47

What is Multi-Region Access Point?

### Answer

A global endpoint that automatically routes requests to the optimal healthy bucket across multiple AWS Regions.

---

# 173. Hands-on Labs

## Lab 26

Host a static website using Amazon S3.

---

## Lab 27

Configure CloudFront in front of an S3 bucket.

---

## Lab 28

Generate a Pre-Signed URL using the AWS CLI or SDK.

---

## Lab 29

Create an Event Notification that triggers a Lambda function when an image is uploaded.

---

## Lab 30

Configure an Access Point for a bucket shared by multiple departments.

---

# 174. One-Page Revision

```
Static Website

↓

CloudFront

↓

Amazon S3

↓

Multipart Upload

↓

Event Notification

↓

Lambda

↓

Object Lock

↓

Access Points

↓

Multi-Region Access Points
```

Remember:

- Static websites → S3 + CloudFront
- Pre-Signed URLs → Temporary private access
- Multipart Upload → Large files (>5 GB)
- Transfer Acceleration → Faster global transfers
- Event Notifications → Automation
- Object Lock → Immutable storage
- Access Points → Simplified access management
- Multi-Region Access Points → Global routing and failover

---

# Think Like a Production Engineer

Advanced S3 features transform simple object storage into a scalable application platform.

In production:

1. Serve websites using **CloudFront + S3**.
2. Never make sensitive buckets public.
3. Use **Pre-Signed URLs** for temporary secure sharing.
4. Use **Multipart Upload** for large objects.
5. Automate workflows with **Event Notifications** and **Lambda**.
6. Protect critical records with **Object Lock**.
7. Simplify permissions using **Access Points**.
8. Improve global performance with **Transfer Acceleration** and **Multi-Region Access Points**.

A mature S3 architecture is **secure, automated, globally scalable, and highly performant**.

# End of Part 6
