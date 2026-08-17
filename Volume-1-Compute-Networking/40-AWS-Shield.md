# AWS Shield

> AWS DevOps Playbook
>
> Volume 1 – Security Services
>
> Chapter 40
>
> AWS Shield (DDoS Protection, Shield Standard, Shield Advanced & Production Architecture)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand DDoS attacks
- Learn AWS Shield
- Understand Shield Standard
- Learn Shield Advanced
- Understand attack detection
- Learn DDoS mitigation
- Understand DDoS Response Team (DRT)
- Learn Cost Protection
- Design Production Architecture
- Answer Interview Questions

---

# 1. What is AWS Shield?

AWS Shield is a managed **Distributed Denial of Service (DDoS)** protection service.

Its purpose is to protect AWS resources from attacks that try to make your application unavailable.

Think of it as AWS's first line of defense against internet-scale attacks.

---

# 2. What is a DDoS Attack?

DDoS stands for

```
Distributed Denial of Service
```

Instead of one attacker,

Thousands or even millions of devices send requests simultaneously.

```
Attacker 1

Attacker 2

Attacker 3

Attacker 4

...

Thousands of Bots

↓

Victim Server
```

The goal is to exhaust resources so legitimate users cannot access the application.

---

# 3. Why Do Attackers Launch DDoS Attacks?

Common reasons include:

- Financial Extortion
- Political Activism
- Revenge
- Competitor Sabotage
- Ransom Demands
- Service Disruption

---

# 4. Without AWS Shield

```
Internet

↓

Millions of Requests

↓

Application Load Balancer

↓

EC2

↓

Application Crash
```

The application becomes unavailable.

---

# 5. With AWS Shield

```
Internet

↓

AWS Shield

↓

Malicious Traffic Filtered

↓

CloudFront

↓

ALB

↓

Application
```

Legitimate traffic continues to reach the application.

---

# 6. Types of DDoS Attacks

Broadly, DDoS attacks fall into three categories:

### Volumetric Attacks

Consume bandwidth.

Examples:

- UDP Flood
- DNS Amplification
- NTP Amplification

---

### Protocol Attacks

Target network infrastructure.

Examples:

- SYN Flood
- ACK Flood
- Ping Flood

---

### Application Layer Attacks

Target the application itself.

Examples:

- HTTP Flood
- Login Flood
- API Abuse

---

# 7. OSI Layers

```
Layer 7

↓

HTTP Flood

↓

WAF
```

```
Layer 3 & 4

↓

Network Flood

↓

AWS Shield
```

This is why WAF and Shield complement each other.

---

# 8. Shield Standard

AWS Shield Standard is automatically enabled for every AWS customer.

You do not need to configure or pay separately for it.

It protects against common network and transport layer attacks.

---

# 9. Services Protected by Shield Standard

Examples include:

- Amazon CloudFront
- Route 53
- Global Accelerator
- Elastic Load Balancing (ALB/NLB/CLB)

These internet-facing services receive baseline DDoS protection.

---

# 10. Shield Advanced

Shield Advanced is a paid service designed for applications requiring enhanced DDoS protection.

It provides:

- Advanced Detection
- Faster Response
- Detailed Visibility
- Cost Protection
- DDoS Response Team (DRT)

---

# 11. Shield Standard vs Shield Advanced

| Shield Standard | Shield Advanced |
|-----------------|-----------------|
| Free | Paid Subscription |
| Automatic | Manual Enrollment |
| Basic DDoS Protection | Advanced DDoS Protection |
| Basic Monitoring | Detailed Metrics & Diagnostics |
| No DRT | DDoS Response Team (DRT) |
| No Cost Protection | Cost Protection Available |

---

# 12. Shield Advanced Architecture

```
Internet

↓

AWS Shield Advanced

↓

CloudFront

↓

AWS WAF

↓

ALB

↓

Auto Scaling

↓

EC2
```

---

# 13. DDoS Detection

AWS continuously monitors incoming traffic.

```
Normal Traffic

↓

Traffic Spike

↓

Traffic Analysis

↓

Attack Detected

↓

Automatic Mitigation
```

---

# 14. Automatic Mitigation

AWS can automatically mitigate many attacks.

```
Attack

↓

Detection

↓

Filtering

↓

Application Continues
```

The goal is to reduce downtime without manual intervention.

---

# 15. DDoS Response Team (DRT)

Shield Advanced customers can engage the AWS DDoS Response Team.

The DRT assists with:

- Live Attack Analysis
- Mitigation Guidance
- Incident Response
- Recovery Assistance

This is especially valuable during large-scale attacks.

---

# 16. Health-Based Detection

Shield Advanced can integrate with application health signals.

```
CloudWatch Alarm

↓

Application Unhealthy

↓

Shield Prioritizes Mitigation
```

This helps AWS distinguish between traffic spikes and actual service impact.

---

# 17. Cost Protection

Imagine an attack generates:

```
500 TB

Traffic
```

This could increase AWS service usage costs.

Shield Advanced includes cost protection for eligible scaling charges resulting from a DDoS attack.

Always review AWS documentation for current eligibility and terms.

---

# 18. Shield + Auto Scaling

```
Attack

↓

Traffic Increases

↓

Auto Scaling

↓

New EC2 Instances

↓

Application Stays Available
```

Shield helps filter attack traffic while Auto Scaling handles legitimate increases in demand.

---

# 19. Shield + CloudFront

One of the most common production architectures.

```
Internet

↓

CloudFront Edge Locations

↓

AWS Shield

↓

WAF

↓

Origin
```

CloudFront helps absorb and distribute traffic globally.

---

# 20. Shield + Route 53

Route 53 is automatically protected by Shield Standard.

```
DNS Query

↓

Shield

↓

Route53

↓

Application
```

---

# 21. Shield + Global Accelerator

```
Users

↓

Global Accelerator

↓

Shield

↓

AWS Network
```

This combination improves availability and resilience.

---

# 22. Shield + Elastic Load Balancer

```
Internet

↓

Shield

↓

ALB

↓

EC2
```

The load balancer benefits from AWS Shield protections.

---

# 23. Shield + WAF

Many people confuse these services.

Shield

```
Protects

↓

DDoS
```

WAF

```
Protects

↓

Web Attacks
```

Together they provide layered security.

---

# 24. Example Attack

Attacker sends

```
10 Million

HTTP Requests
```

Shield

↓

Absorbs and mitigates network-scale attack traffic.

WAF

↓

Applies application-layer rules such as rate limiting and SQL Injection detection.

Application

↓

Continues Serving Legitimate Users

---

# 25. Real Production Architecture

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

Auto Scaling

↓

EC2

↓

Amazon RDS
```

---

# 26. Enterprise Example

An online banking application.

Requirements

- 24×7 Availability
- Global Users
- DDoS Protection
- SQL Injection Protection
- XSS Protection

Solution

```
AWS Shield Advanced

↓

CloudFront

↓

AWS WAF

↓

ALB

↓

Auto Scaling

↓

Multi-AZ RDS
```

---

# 27. Best Practices

✔ Use CloudFront for internet-facing applications.

✔ Combine Shield with AWS WAF.

✔ Enable Auto Scaling.

✔ Use Route 53 for DNS resilience.

✔ Monitor CloudWatch metrics.

✔ Use Shield Advanced for mission-critical workloads.

✔ Develop and test an incident response plan.

---

# 28. Common Mistakes

❌ Assuming WAF alone stops all DDoS attacks.

❌ Ignoring network-layer attacks.

❌ Not using CloudFront.

❌ No Auto Scaling.

❌ No monitoring.

❌ Not understanding the difference between Shield Standard and Advanced.

---

# 29. Interview Questions

## Question 1

What is AWS Shield?

### Answer

AWS Shield is a managed DDoS protection service that helps protect AWS resources against network and transport layer attacks.

---

## Question 2

What is the difference between Shield Standard and Shield Advanced?

### Answer

Shield Standard provides automatic baseline DDoS protection at no additional cost for supported AWS services. Shield Advanced is a paid offering that provides enhanced detection, additional visibility, DDoS Response Team (DRT) support, and cost protection for eligible scaling charges.

---

## Question 3

Is Shield Standard free?

### Answer

Yes.

Shield Standard is automatically available for all AWS customers.

---

## Question 4

Which AWS services are commonly protected by Shield Standard?

### Answer

Examples include CloudFront, Route 53, Global Accelerator, and Elastic Load Balancing.

---

## Question 5

What is DRT?

### Answer

DRT stands for DDoS Response Team, an AWS team available to Shield Advanced customers for assistance during DDoS incidents.

---

## Question 6

Can Shield replace AWS WAF?

### Answer

No.

Shield protects primarily against DDoS attacks, while AWS WAF protects web applications from Layer 7 threats such as SQL Injection and Cross-Site Scripting.

---

## Question 7

Why should Shield and WAF be used together?

### Answer

Shield mitigates network and transport layer DDoS attacks, while WAF inspects HTTP/HTTPS requests and blocks application-layer attacks. Together they provide defense in depth.

---

## Question 8

Does Shield automatically stop every attack?

### Answer

Shield automatically mitigates many DDoS attacks, but no security service can guarantee protection against every possible attack. Layered security, monitoring, and incident response remain important.

---

# 30. Hands-on Labs

## Lab 1

Identify which AWS resources in your account are internet-facing and automatically protected by Shield Standard.

---

## Lab 2

Deploy an Application Load Balancer with CloudFront and AWS WAF, then diagram where Shield operates.

---

## Lab 3

Create a CloudWatch dashboard to monitor traffic and request rates during load testing.

---

## Lab 4

Compare the features of Shield Standard and Shield Advanced for a production application.

---

# 31. One-Page Revision

```
AWS Shield

↓

DDoS Protection

↓

Shield Standard

↓

Shield Advanced

↓

Network Attacks

↓

Automatic Mitigation

↓

DRT

↓

Cost Protection

↓

CloudFront

↓

Route53

↓

ALB

↓

Global Accelerator

↓

AWS WAF

↓

High Availability
```

---

# AWS WAF vs AWS Shield vs Security Groups

| Feature | AWS Shield | AWS WAF | Security Group |
|----------|------------|----------|----------------|
| Purpose | DDoS Protection | Web Application Protection | Instance Firewall |
| OSI Layer | Layer 3 & 4 (with additional protections for supported services) | Layer 7 | Layer 4 |
| Protects Against | Network floods, protocol attacks | SQLi, XSS, bots, HTTP floods | Port and protocol access |
| Works With | CloudFront, Route 53, ALB, Global Accelerator | CloudFront, ALB, API Gateway, AppSync | EC2, ENIs |
| Understands HTTP | No | Yes | No |
| Inspects URL/Cookies | No | Yes | No |

---

# Think Like a Principal Cloud Security Architect

AWS Shield is your **first defensive layer** against large-scale DDoS attacks.

A production-grade internet-facing architecture typically looks like this:

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

Each service has a specific responsibility:

- **Shield** keeps the service available during DDoS attacks.
- **CloudFront** distributes and absorbs global traffic.
- **WAF** blocks malicious web requests.
- **Security Groups** enforce network access controls.
- **Auto Scaling** handles legitimate traffic growth.

Rather than relying on a single security service, enterprise AWS environments use **multiple complementary layers** to achieve high availability, strong security, and resilience.

# End of AWS Shield