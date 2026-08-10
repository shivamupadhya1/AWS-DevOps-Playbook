# Amazon Route 53

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 07
>
> Part 1

---

# Chapter Objective

After this chapter you should be able to

- Explain DNS from scratch
- Understand how Route53 resolves requests
- Explain Hosted Zones
- Understand Record Types
- Explain Alias vs CNAME
- Answer Route53 interview questions

---

# 1. Business Problem

Imagine your application is running on

```
54.210.120.55
```

Question

Will users remember

```
54.210.120.55
```

or

```
amazon.com
```

Obviously

```
amazon.com
```

Humans remember names.

Computers communicate using IP addresses.

We need something that converts

```
amazon.com

↓

54.210.120.55
```

AWS provides

Route53.

---

# 2. What is Route53?

Route53 is AWS's managed DNS service.

DNS means

```
Domain Name System
```

Its job is

```
Domain Name

↓

IP Address
```

Example

```
www.company.com

↓

ALB

↓

EC2
```

---

# 3. Why the Name Route53?

Very common interview question.

53

represents

```
Port 53
```

which is the standard DNS port.

Both

```
UDP 53

TCP 53
```

are used by DNS.

---

# 4. Internal Request Flow

Suppose user opens

```
www.company.com
```

Flow

```
Browser

↓

ISP DNS Resolver

↓

Root DNS

↓

TLD (.com)

↓

Route53 Hosted Zone

↓

ALB

↓

EC2

↓

Application
```

---

# 5. Hosted Zones

Hosted Zone

is a container

that stores DNS records.

Example

```
company.com

↓

A Record

↓

MX Record

↓

TXT Record

↓

CNAME

↓

Alias
```

Without Hosted Zone

Route53 cannot answer queries.

---

# 6. Public Hosted Zone

Used for

Internet-facing applications.

Example

```
company.com
```

Accessible from anywhere.

---

# 7. Private Hosted Zone

Accessible only inside

VPC.

Example

```
db.internal.company

↓

Private IP
```

Not accessible from Internet.

---

# 8. DNS Records

---

## A Record

Maps

```
Domain

↓

IPv4
```

Example

```
app.company.com

↓

54.32.15.10
```

---

## AAAA Record

Maps

```
Domain

↓

IPv6
```

---

## CNAME

Maps

```
One Domain

↓

Another Domain
```

Example

```
api.company.com

↓

alb.amazonaws.com
```

Cannot be used for the root domain.

---

## Alias Record

AWS Special Record.

Example

```
company.com

↓

ALB
```

Advantages

- Works with root domain
- Free DNS queries for AWS targets
- Automatically tracks AWS resource IP changes

---

## MX Record

Mail Servers.

---

## TXT Record

Verification

SPF

DKIM

DMARC

Google verification

etc.

---

# 9. Alias vs CNAME

⭐⭐⭐⭐⭐

Most asked interview topic.

| Alias | CNAME |
|---------|---------|
| AWS specific | Standard DNS |
| Root Domain Supported | Root Domain Not Supported |
| Free for AWS targets | Standard DNS queries |
| Tracks ALB changes | Points to hostname |

---

Perfect Interview Answer

Use Alias for AWS resources like ALB, CloudFront, API Gateway, or S3 website endpoints because it supports the zone apex (root domain) and automatically adapts to AWS infrastructure changes.

---

# 10. Production Architecture

```
User

↓

Route53

↓

ALB

↓

Target Group

↓

ASG

↓

EC2

↓

Application
```

---

# 11. Production Scenario

Problem

Application migrated

Old ALB

↓

New ALB

Question

How do users automatically reach the new environment?

Answer

Update the Alias record to point to the new ALB.

No client configuration changes are required.

---

# 12. Best Practices

- Use Alias records for AWS resources.
- Use Private Hosted Zones for internal services.
- Protect your domain registrar account.
- Enable DNSSEC if applicable.
- Keep records documented.

---

# 13. Common Mistakes

❌ Using CNAME at the root domain.

Use Alias instead.

---

❌ Hardcoding ALB IP addresses.

ALB IPs can change.

Always use DNS.

---

# 14. Interview Questions

---

## Question 1

What is Route53?

### Perfect Answer

Amazon Route53 is AWS's highly available managed DNS service that resolves domain names into IP addresses or AWS resources such as ALBs, CloudFront distributions, API Gateway endpoints, and more.

---

## Question 2

Why is it called Route53?

### Perfect Answer

It is named after DNS port 53, which is the standard port used for DNS over UDP and TCP.

---

## Question 3

Difference between Alias and CNAME?

### Perfect Answer

Alias is an AWS-specific record that supports root domains and AWS resources like ALBs and CloudFront. CNAME is a standard DNS record that maps one hostname to another but cannot be used at the zone apex.

---

## Question 4

What is a Hosted Zone?

### Perfect Answer

A Hosted Zone is a container for DNS records associated with a domain. Route53 uses these records to answer DNS queries.

---

## Question 5

Why shouldn't you point users directly to an ALB DNS name?

### Perfect Answer

You generally expose your own domain (for example, `www.company.com`) and map it to the ALB using an Alias record. This keeps branding consistent and lets you change the underlying AWS resource without changing the public URL.

---

# Revision Sheet

Remember

- DNS
- Hosted Zone
- Alias
- CNAME
- Public Hosted Zone
- Private Hosted Zone
- A Record
- AAAA Record
- MX
- TXT

```
User

↓

DNS Query

↓

Route53

↓

Alias

↓

ALB

↓

Application
```

---

# End of Part 1
