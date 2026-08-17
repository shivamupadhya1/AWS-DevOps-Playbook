# AWS WAF – Part 1

> AWS DevOps Playbook
>
> Volume 1 – Security Services
>
> Chapter 39
>
> AWS WAF – Part 1 (Introduction, Web Attacks, Web ACLs & Rule Engine)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand AWS WAF
- Understand Layer 7 Security
- Learn common Web Attacks
- Understand SQL Injection (SQLi)
- Learn Cross-Site Scripting (XSS)
- Understand Web ACL
- Learn Rules and Rule Groups
- Learn IP Sets
- Understand Request Flow
- Design Production WAF Architecture
- Answer Interview Questions

---

# 1. What is AWS WAF?

AWS WAF (Web Application Firewall) is a managed firewall that protects **web applications** from common internet attacks.

Unlike Security Groups or Network ACLs, WAF understands **HTTP and HTTPS requests**.

It can inspect:

- URLs
- Headers
- Cookies
- Query Strings
- HTTP Methods
- Request Body (within supported inspection limits)

---

# 2. Why Do We Need WAF?

Imagine your application is available on the internet.

```
Internet

↓

Millions of Users

↓

Application
```

Not everyone is a legitimate user.

Some requests are:

- SQL Injection
- Cross Site Scripting
- Bot Requests
- HTTP Flood
- Malicious Crawlers

AWS WAF filters these requests before they reach your application.

---

# 3. Without WAF

```
Internet

↓

Hackers

↓

ALB

↓

Application

↓

Database
```

The application receives every request.

---

# 4. With WAF

```
Internet

↓

AWS WAF

↓

ALB

↓

Application
```

Bad requests are blocked before reaching the application.

---

# 5. OSI Layer

AWS WAF operates at:

```
Layer 7

(Application Layer)
```

It understands HTTP/HTTPS traffic.

---

# 6. WAF vs Security Group

| AWS WAF | Security Group |
|----------|----------------|
| Layer 7 | Layer 4 |
| HTTP/HTTPS | TCP/UDP |
| URLs | Ports |
| Cookies | IP/Port |
| Headers | Source/Destination |
| SQLi Protection | No |

---

# 7. WAF vs NACL

| WAF | Network ACL |
|------|-------------|
| Application Layer | Network Layer |
| Understands HTTP | Only IP & Ports |
| Managed Service | VPC Component |
| Blocks SQLi | Cannot detect SQLi |

---

# 8. AWS Services Supported

AWS WAF can protect:

- Amazon CloudFront
- Application Load Balancer (ALB)
- Amazon API Gateway
- AWS AppSync
- Amazon Cognito (certain integrations)
- AWS Verified Access

It **cannot** be directly attached to EC2 instances.

---

# 9. Real Production Architecture

```
Internet

↓

AWS Shield

↓

CloudFront

↓

AWS WAF

↓

ALB

↓

EC2 / ECS / EKS

↓

Database
```

This is a common enterprise architecture.

---

# 10. What Can WAF Inspect?

AWS WAF can inspect:

- URI Path
- Query String
- Cookies
- HTTP Headers
- Request Body (size limits apply)
- Source IP
- Country
- HTTP Method

Example

```
GET

/login

?id=100
```

Every component can be evaluated by WAF rules.

---

# 11. Common Web Attacks

AWS WAF protects against:

- SQL Injection (SQLi)
- Cross Site Scripting (XSS)
- HTTP Flood
- Bot Traffic
- Bad IP Addresses
- Geographic Attacks
- Credential Stuffing (with additional managed features)
- Known Vulnerabilities (through managed rule groups)

---

# 12. SQL Injection (SQLi)

Suppose an application executes:

```sql
SELECT * FROM users
WHERE username='admin'
AND password='password';
```

An attacker enters:

```sql
' OR 1=1 --
```

The query becomes:

```sql
SELECT * FROM users
WHERE username=''
OR 1=1;
```

Now the attacker may bypass authentication if the application is vulnerable.

AWS WAF helps detect and block common SQLi patterns before they reach the application.

---

# 13. Cross Site Scripting (XSS)

Suppose a comment box accepts:

```html
<script>

alert("Hacked")

</script>
```

If the application fails to sanitize the input:

```
User Browser

↓

Malicious Script Executes
```

AWS WAF can detect and block common XSS payloads.

---

# 14. HTTP Flood

Attackers send thousands of HTTP requests.

```
Attacker

↓

50,000 Requests

↓

Application
```

WAF can rate-limit or block excessive requests.

---

# 15. Bot Traffic

Examples:

- Web Scrapers
- Fake Browsers
- Spam Bots
- Vulnerability Scanners

AWS WAF can help identify and block unwanted automated traffic.

---

# 16. Geographic Blocking

Example

```
Application

Only India

Allowed
```

Requests from other countries can be blocked if required.

Useful for region-specific applications.

---

# 17. What is a Web ACL?

A Web ACL (Access Control List) is the primary resource in AWS WAF.

It contains:

```
Web ACL

│

├── Rules

├── Rule Groups

├── Default Action

└── Associated Resources
```

Think of it as the "policy" attached to your application.

---

# 18. Default Action

Every Web ACL must define a default action.

Options

```
Allow
```

or

```
Block
```

Typical production approach:

```
Allow

↓

Block only malicious requests
```

---

# 19. Rule

A Rule defines what AWS WAF should inspect.

Example

```
If

Country = China

↓

Block
```

Another

```
If

URI=/admin

↓

Allow only Corporate IP
```

---

# 20. Rule Priority

Rules are evaluated in order.

Example

```
Rule 1

↓

Rule 2

↓

Rule 3

↓

Default Action
```

The first matching rule determines the action.

---

# 21. Rule Actions

A rule can:

- Allow
- Block
- Count (for monitoring)
- CAPTCHA
- Challenge

The **Count** action is useful for testing before enforcing a rule.

---

# 22. Rule Groups

Instead of creating many individual rules:

```
100 Rules
```

Use:

```
Rule Group

↓

Reusable Rules
```

Rule Groups simplify management.

---

# 23. AWS Managed Rule Groups

AWS provides managed rule groups for common threats.

Examples include protection against:

- SQL Injection
- Cross Site Scripting
- Known Bad Inputs
- Linux-specific attacks
- PHP-specific attacks

AWS updates these rule sets regularly.

---

# 24. IP Sets

An IP Set is a collection of IP addresses.

Example

```
Office IP

↓

10.10.10.10

↓

20.20.20.20
```

Rule

```
Only

Office IP

↓

Allow
```

---

# 25. Request Evaluation Flow

```
Internet

↓

WAF

↓

Web ACL

↓

Rule 1

↓

Rule 2

↓

Rule 3

↓

Default Action

↓

Application
```

---

# 26. Real Production Example

Suppose

```
/admin
```

should only be accessible from corporate offices.

Rule

```
URI=/admin

AND

Source IP = Office

↓

Allow

Else

↓

Block
```

---

# 27. Best Practices

✔ Use AWS Managed Rule Groups.

✔ Start new rules in **Count** mode before switching to **Block**.

✔ Keep rule priorities organized.

✔ Restrict admin endpoints.

✔ Use IP Sets for office access.

✔ Regularly review blocked requests.

---

# 28. Common Mistakes

❌ Blocking legitimate traffic without testing.

❌ Ignoring rule order.

❌ Not updating custom rules.

❌ Allowing unrestricted access to admin pages.

❌ Depending only on WAF without secure application coding.

---

# 29. Interview Questions

## Question 1

What is AWS WAF?

### Answer

AWS WAF is a managed Web Application Firewall that protects web applications from common Layer 7 attacks such as SQL Injection, Cross-Site Scripting, bots, and malicious HTTP requests.

---

## Question 2

At which OSI layer does AWS WAF operate?

### Answer

Layer 7 (Application Layer).

---

## Question 3

What is a Web ACL?

### Answer

A Web ACL is the primary AWS WAF resource that contains rules, rule groups, default actions, and associations with protected resources.

---

## Question 4

What is the difference between AWS WAF and a Security Group?

### Answer

AWS WAF inspects HTTP/HTTPS requests at Layer 7, while Security Groups filter TCP/UDP traffic at Layer 4 based on IP addresses and ports.

---

## Question 5

Can AWS WAF protect EC2 instances directly?

### Answer

No.

AWS WAF protects supported services such as CloudFront, Application Load Balancers, API Gateway, AppSync, Cognito (supported integrations), and Verified Access. It is not attached directly to EC2 instances.

---

## Question 6

What is an IP Set?

### Answer

An IP Set is a reusable collection of IP addresses that can be referenced by WAF rules to allow or block traffic.

---

## Question 7

Why should new WAF rules initially use the Count action?

### Answer

The Count action lets you observe how a rule would affect traffic without blocking requests, helping reduce false positives before enforcement.

---

# 30. Hands-on Labs

## Lab 1

Create a Web ACL.

---

## Lab 2

Associate the Web ACL with an Application Load Balancer.

---

## Lab 3

Create an IP Set containing your office IP addresses.

---

## Lab 4

Create a rule that blocks traffic from a selected country.

---

## Lab 5

Enable the AWS Managed Core Rule Set and monitor requests using the **Count** action.

---

# 31. One-Page Revision

```
AWS WAF

↓

Layer 7 Firewall

↓

Web ACL

↓

Rules

↓

Rule Groups

↓

IP Sets

↓

Managed Rules

↓

SQL Injection

↓

Cross Site Scripting

↓

Bot Protection

↓

CloudFront

↓

ALB

↓

API Gateway
```

---

# Think Like a Senior DevSecOps Engineer

Security Groups protect **instances**.

Network ACLs protect **subnets**.

AWS WAF protects **web applications**.

A production-ready internet-facing application typically uses multiple layers:

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

Security Groups

↓

Application

↓

Database
```

Each layer blocks different types of threats. Never rely on a single security mechanism—**defense in depth** is the key principle for securing production workloads.

# End of Part 1