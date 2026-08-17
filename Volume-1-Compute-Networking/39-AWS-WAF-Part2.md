# AWS WAF – Part 2

> AWS DevOps Playbook
>
> Volume 1 – Security Services
>
> Chapter 39
>
> AWS WAF – Part 2 (Advanced Rules, Rate Limiting, Bot Protection, Logging & Production Best Practices)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand Managed Rule Groups
- Learn Custom Rules
- Learn Rate Limiting
- Understand CAPTCHA & Challenge
- Learn Geo Match
- Understand Bot Control
- Learn Logging
- Integrate WAF with CloudWatch
- Learn WAF Pricing
- Design Production WAF Architecture
- Answer Interview Questions

---

# 32. Managed Rule Groups

Instead of creating every rule manually, AWS provides pre-built rule groups.

Examples include:

- AWS Core Rule Set (CRS)
- Known Bad Inputs
- SQL Injection
- Linux Rules
- Unix Rules
- PHP Rules
- WordPress Rules
- Admin Protection

Architecture

```
AWS WAF

↓

Managed Rule Group

↓

Updated by AWS

↓

Application Protected
```

---

# 33. Why Managed Rules?

Imagine a new SQL Injection technique is discovered.

Without Managed Rules

```
Security Engineer

↓

Update Rules

↓

Deploy
```

With Managed Rules

```
AWS Updates

↓

Protection Applied
```

This reduces operational effort.

---

# 34. Custom Rules

Sometimes every application has different requirements.

Example

```
Allow

/company

↓

Only Corporate IP
```

Or

```
Block

/admin

↓

Outside India
```

Custom Rules help implement business-specific security policies.

---

# 35. Rule Statements

Rules can inspect:

- URI Path
- Query String
- Cookies
- Headers
- Body
- Source IP
- Country
- HTTP Method
- Labels

Example

```
POST

/admin/login

↓

Inspect
```

---

# 36. Combining Conditions

AWS WAF supports logical operators.

```
AND

OR

NOT
```

Example

```
Country = India

AND

URI = /admin

↓

Allow
```

---

# 37. Rate-Based Rules

One of the most useful WAF features.

Example

```
Normal User

↓

10 Requests

↓

Allowed
```

Attacker

```
5000 Requests

↓

Blocked
```

---

# 38. Rate Limiting Example

Rule

```
More than

2000 Requests

per 5 Minutes

↓

Block
```

This helps mitigate HTTP Flood attacks.

---

# 39. CAPTCHA

Instead of immediately blocking users,

AWS WAF can ask them to solve a CAPTCHA.

Example

```
User

↓

CAPTCHA

↓

Human?

↓

Allow
```

Bots usually fail this verification.

---

# 40. Challenge

Challenge is lighter than CAPTCHA.

```
Browser

↓

Challenge

↓

JavaScript Verification

↓

Allow
```

Useful for filtering automated traffic with less impact on legitimate users.

---

# 41. Geo Match Rules

Suppose your application is only for India.

Rule

```
India

↓

Allow

Else

↓

Block
```

---

# 42. IP Reputation Lists

AWS maintains IP reputation intelligence.

Example

```
Known Malicious IP

↓

Automatically Block
```

This protects against known attack sources.

---

# 43. Anonymous IP Protection

AWS can identify traffic from:

- VPNs
- TOR Exit Nodes
- Anonymous Proxies

Example

```
TOR

↓

Blocked
```

---

# 44. Bot Control

Bots are everywhere.

Examples

- Search Crawlers
- Price Scrapers
- Credential Stuffers
- Fake Browsers

AWS WAF Bot Control identifies and manages these bots.

---

# 45. Bot Categories

```
Good Bots

↓

Google

↓

Bing

↓

Allowed
```

```
Bad Bots

↓

Spam

↓

Scrapers

↓

Blocked
```

---

# 46. Fraud Control

AWS also offers specialized managed protections for:

- Account Creation Fraud
- Account Takeover Attempts

These features help protect authentication workflows.

---

# 47. Labels

Rules can assign labels to requests.

Example

```
SQL Injection

↓

Label Added

↓

Another Rule

↓

Block
```

Labels make complex rule logic easier to manage.

---

# 48. Custom Responses

Instead of returning:

```
403 Forbidden
```

You can return:

```
Access Denied

Contact Support
```

Or redirect users to another page.

---

# 49. Logging

AWS WAF logs every inspected request (when enabled).

Architecture

```
WAF

↓

Logs

↓

CloudWatch Logs

or

Amazon S3

or

Amazon Kinesis Data Firehose
```

These logs are useful for investigations and tuning.

---

# 50. What Gets Logged?

Each log can include:

- Timestamp
- Client IP
- Country
- URI
- HTTP Method
- Rule Matched
- Action (Allow/Block/Count)
- Headers
- Labels

---

# 51. CloudWatch Integration

```
AWS WAF

↓

CloudWatch

↓

Metrics

↓

Alarms
```

Examples:

- Blocked Requests
- Allowed Requests
- CAPTCHA Solved
- Rate-Limited Requests

---

# 52. CloudWatch Dashboard

Monitor:

- Total Requests
- Allowed Requests
- Blocked Requests
- Top IP Addresses
- Top Countries
- Top Rules Triggered

This helps identify attack patterns.

---

# 53. WAF Metrics

Common metrics include:

```
Allowed Requests

Blocked Requests

Counted Requests

CAPTCHA Requests
```

These metrics can be visualized in CloudWatch.

---

# 54. WAF Pricing

Pricing depends on:

- Number of Web ACLs
- Number of Rules
- Number of Requests
- Managed Rule Groups
- Bot Control
- CAPTCHA usage

Design rule sets carefully to control costs.

---

# 55. Production Architecture

```
Internet

↓

AWS Shield

↓

CloudFront

↓

AWS WAF

↓

Application Load Balancer

↓

Auto Scaling

↓

EC2 / ECS / EKS

↓

Amazon RDS
```

---

# 56. Enterprise Example

An e-commerce company:

Requirements

- Stop SQL Injection
- Stop XSS
- Prevent HTTP Floods
- Block Malicious Bots
- Allow Google Search Bots

Solution

```
AWS Managed Rules

↓

Bot Control

↓

Rate Limiting

↓

Geo Match

↓

IP Reputation

↓

CAPTCHA
```

---

# 57. Best Practices

✔ Enable AWS Managed Rule Groups.

✔ Use **Count** mode before switching to **Block**.

✔ Enable logging for analysis.

✔ Use rate-based rules for public APIs.

✔ Protect admin URLs separately.

✔ Use IP Sets for trusted networks.

✔ Review CloudWatch metrics regularly.

✔ Keep custom rules simple and well documented.

---

# 58. Common Mistakes

❌ Blocking search engine crawlers.

❌ Using too many overlapping rules.

❌ Ignoring WAF logs.

❌ No rate limiting.

❌ No monitoring.

❌ Applying WAF only after an attack occurs.

---

# 59. Interview Questions

## Question 8

What are AWS Managed Rule Groups?

### Answer

They are AWS-maintained security rule sets that protect against common web attacks such as SQL Injection, Cross-Site Scripting, and known malicious inputs.

---

## Question 9

What is a Rate-Based Rule?

### Answer

A Rate-Based Rule tracks the number of requests from a source over a defined time window and can block or limit traffic that exceeds the configured threshold.

---

## Question 10

What is the difference between CAPTCHA and Challenge?

### Answer

CAPTCHA requires user interaction to prove the requester is human, while Challenge performs browser-based verification with minimal user interaction.

---

## Question 11

What is Bot Control?

### Answer

Bot Control is an AWS WAF managed capability that identifies and manages automated traffic, allowing legitimate bots while blocking or challenging malicious ones.

---

## Question 12

Can AWS WAF generate CloudWatch metrics?

### Answer

Yes. AWS WAF publishes metrics such as allowed requests, blocked requests, and rate-limited requests, which can be monitored with CloudWatch.

---

## Question 13

Can AWS WAF replace secure application coding?

### Answer

No.

AWS WAF provides an additional layer of protection, but secure coding, input validation, authentication, and regular security testing are still essential.

---

## Question 14

Where are AWS WAF logs stored?

### Answer

AWS WAF logs can be sent to CloudWatch Logs, Amazon S3 (via Kinesis Data Firehose), or Amazon Kinesis Data Firehose destinations for analysis.

---

## Question 15

How would you protect a login page from brute-force attacks?

### Answer

Use a combination of Rate-Based Rules, Bot Control, CAPTCHA/Challenge, IP reputation lists, and monitoring with CloudWatch.

---

# 60. Hands-on Labs

## Lab 6

Enable the AWS Managed Core Rule Set.

---

## Lab 7

Create a Rate-Based Rule that blocks IPs sending more than 2,000 requests in 5 minutes.

---

## Lab 8

Enable Bot Control and observe detected bot traffic.

---

## Lab 9

Enable WAF logging and review blocked requests.

---

## Lab 10

Create a CloudWatch Alarm when blocked requests exceed a threshold.

---

# 61. One-Page Revision

```
AWS WAF

↓

Managed Rules

↓

Custom Rules

↓

Rate Limiting

↓

Bot Control

↓

CAPTCHA

↓

Challenge

↓

Geo Match

↓

IP Reputation

↓

Logging

↓

CloudWatch

↓

CloudFront

↓

ALB
```

---

# Think Like a Senior DevSecOps Engineer

AWS WAF is your **application-layer security gatekeeper**.

In production:

1. Place **AWS Shield** in front for DDoS protection.
2. Use **CloudFront** to distribute traffic globally.
3. Attach **AWS WAF** to inspect every HTTP/HTTPS request.
4. Enable **Managed Rule Groups** for baseline protection.
5. Add **Rate-Based Rules** to mitigate brute-force and HTTP flood attacks.
6. Use **Bot Control** and **CAPTCHA** for automated threats.
7. Enable **logging and CloudWatch metrics** to continuously monitor and tune your rules.

A well-designed WAF should **block malicious traffic while allowing legitimate users with minimal impact**, balancing security, availability, and user experience.

# End of Part 2