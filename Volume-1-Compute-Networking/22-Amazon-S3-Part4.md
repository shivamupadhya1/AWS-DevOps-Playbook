# Amazon S3

> AWS DevOps Playbook
>
> Volume 1 – Storage
>
> Chapter 22
>
> Amazon S3 – Part 4 (Encryption)

---

# Chapter Objectives

After completing this chapter, you should be able to:

- Understand why encryption is important
- Learn Server-Side Encryption (SSE)
- Understand SSE-S3
- Learn SSE-KMS
- Learn SSE-C
- Understand Client-Side Encryption
- Learn Envelope Encryption
- Understand KMS Keys
- Learn Bucket Keys
- Understand encryption best practices
- Answer interview questions

---

# 83. Why Do We Need Encryption?

Suppose someone steals a storage disk.

Without encryption:

```
Disk

↓

Read Data

↓

Sensitive Information Exposed
```

With encryption:

```
Disk

↓

Encrypted Data

↓

Unreadable

↓

Without Key
```

Encryption protects your data even if someone gains unauthorized access to the storage media.

---

# 84. Encryption at Rest vs Encryption in Transit

There are two major types of encryption.

### Encryption at Rest

Protects stored data.

```
Object

↓

Encrypted

↓

Stored in Amazon S3
```

Example:

- Images
- Videos
- Documents
- Database backups

---

### Encryption in Transit

Protects data while moving over the network.

```
Laptop

↓

HTTPS (TLS)

↓

Amazon S3
```

Always use HTTPS when accessing Amazon S3.

---

# 85. Server-Side Encryption (SSE)

In Server-Side Encryption:

```
Application

↓

Upload File

↓

Amazon S3

↓

Encrypt Object

↓

Store Encrypted Data
```

AWS encrypts the object after receiving it.

There are three Server-Side Encryption methods:

- SSE-S3
- SSE-KMS
- SSE-C

---

# 86. SSE-S3

SSE-S3 means:

**Server-Side Encryption using Amazon S3 managed keys.**

AWS manages:

- Encryption keys
- Key rotation
- Key storage
- Encryption process

```
Application

↓

Upload

↓

S3

↓

AWS Managed Key

↓

Encrypted Object
```

This is the simplest encryption option.

---

# 87. Characteristics of SSE-S3

✔ AWS manages everything

✔ No key management

✔ Simple configuration

✔ Good performance

✔ AES-256 encryption

Suitable for:

- General applications
- Website assets
- Documents
- Images

---

# 88. SSE-S3 Example

Suppose an application uploads:

```
invoice.pdf
```

Flow:

```
Application

↓

S3

↓

AWS Encryption

↓

Encrypted Object Stored
```

The application does not need to manage encryption keys.

---

# 89. SSE-KMS

SSE-KMS uses:

**AWS Key Management Service (KMS)**

instead of S3-managed keys.

```
Application

↓

Amazon S3

↓

AWS KMS

↓

Encryption Key

↓

Encrypted Object
```

You control the KMS key policies and permissions.

---

# 90. Advantages of SSE-KMS

✔ Centralized key management

✔ Audit logs through AWS CloudTrail

✔ Fine-grained IAM permissions

✔ Customer Managed Keys (CMKs)

✔ Automatic or manual key rotation

✔ Compliance support

---

# 91. When Should You Use SSE-KMS?

Examples:

- Banking
- Healthcare
- Government
- Financial records
- HR documents
- Compliance workloads

If you need to control who can use encryption keys, SSE-KMS is usually the better choice.

---

# 92. AWS Managed Key vs Customer Managed Key

AWS KMS provides different types of keys.

### AWS Managed Key

```
AWS

↓

Creates Key

↓

Manages Key
```

Simple to use.

Less administrative control.

---

### Customer Managed Key (CMK)

```
You

↓

Create Key

↓

Manage Permissions

↓

Rotate

↓

Delete

↓

Audit
```

Provides maximum control.

---

# 93. SSE-S3 vs SSE-KMS

| Feature | SSE-S3 | SSE-KMS |
|----------|---------|----------|
| Key Managed By | Amazon S3 | AWS KMS |
| Customer Control | No | Yes |
| CloudTrail Key Usage Logs | No | Yes |
| Fine-Grained Permissions | No | Yes |
| Compliance Support | Good | Excellent |

---

# 94. SSE-C

SSE-C means:

**Server-Side Encryption with Customer-Provided Keys**

Flow:

```
Application

↓

Uploads File

+

Encryption Key

↓

Amazon S3

↓

Encrypt

↓

Discard Key
```

AWS never stores the customer-provided key.

The client must provide the same key for future object access.

---

# 95. Characteristics of SSE-C

Advantages:

- Customer controls encryption keys
- AWS does not retain the keys

Disadvantages:

- Complex key management
- Risk of permanent data loss if the key is lost
- Less commonly used than SSE-S3 or SSE-KMS

---

# 96. Client-Side Encryption

Here, the application encrypts the data **before** sending it to Amazon S3.

```
Application

↓

Encrypt

↓

Encrypted File

↓

Upload

↓

Amazon S3
```

Amazon S3 stores the encrypted object but never sees the plaintext.

---

# 97. Comparison of Encryption Methods

| Method | Who Encrypts? | Who Manages Keys? |
|----------|---------------|-------------------|
| SSE-S3 | Amazon S3 | Amazon S3 |
| SSE-KMS | Amazon S3 | AWS KMS / Customer |
| SSE-C | Amazon S3 | Customer |
| Client-Side Encryption | Client Application | Customer |

---

# 98. Envelope Encryption

Envelope Encryption uses two keys.

```
Data

↓

Encrypted Using

↓

Data Key

↓

Data Key Encrypted Using

↓

KMS Key
```

This approach combines security with performance and is widely used in AWS encryption services.

---

# 99. Bucket Keys

Normally:

```
S3

↓

KMS

↓

Every Request
```

With **S3 Bucket Keys**:

```
Bucket

↓

Bucket Key

↓

Fewer KMS Requests

↓

Lower Cost
```

Benefits:

- Reduced KMS API calls
- Lower KMS costs
- Improved scalability for large workloads

---

# 100. Default Bucket Encryption

You can configure a bucket so that every uploaded object is automatically encrypted.

Example:

```
Bucket

↓

Default Encryption

↓

SSE-KMS
```

Users do not need to specify encryption for every upload.

---

# 101. Production Example

A bank stores customer statements.

Requirements:

- Encryption
- Auditing
- Compliance

Architecture:

```
Application

↓

Amazon S3

↓

SSE-KMS

↓

Customer Managed Key

↓

CloudTrail Logs
```

This allows both encryption and auditing of key usage.

---

# 102. Best Practices

✔ Enable default bucket encryption.

✔ Use SSE-KMS for sensitive data.

✔ Rotate customer-managed keys when required.

✔ Restrict KMS permissions using IAM.

✔ Use HTTPS for all uploads and downloads.

✔ Enable CloudTrail for auditing KMS key usage.

✔ Use S3 Bucket Keys when appropriate to reduce KMS costs.

---

# 103. Common Mistakes

❌ Uploading sensitive data without encryption.

❌ Giving unrestricted access to KMS keys.

❌ Losing customer-provided keys (SSE-C).

❌ Using HTTP instead of HTTPS.

❌ Disabling key rotation where required by policy.

---

# 104. Production Scenario 1

### Problem

A compliance team requires:

- Encryption
- Audit trail
- Controlled key access

Solution:

- SSE-KMS
- Customer Managed Key
- CloudTrail
- IAM policies on the KMS key

---

# 105. Production Scenario 2

### Problem

Millions of encrypted uploads generate high KMS costs.

Solution:

Enable **S3 Bucket Keys** to reduce KMS API requests.

---

# 106. Production Scenario 3

### Problem

A developer accidentally uploads unencrypted objects.

Solution:

Enable **Default Bucket Encryption** and enforce encryption using a Bucket Policy if required.

---

# 107. Production Scenario 4

### Problem

A customer loses the encryption key used with SSE-C.

Result:

The encrypted objects cannot be decrypted.

Lesson:

Carefully manage customer-provided keys or use SSE-KMS where appropriate.

---

# 108. Interview Questions

## Question 24

What are the different server-side encryption options in Amazon S3?

### Answer

- SSE-S3
- SSE-KMS
- SSE-C

---

## Question 25

Which encryption method gives the customer the most control?

### Answer

SSE-C and Client-Side Encryption provide the highest level of customer control over encryption keys. SSE-KMS with customer-managed keys provides strong control while allowing AWS KMS to manage key storage.

---

## Question 26

What is the difference between SSE-S3 and SSE-KMS?

### Answer

SSE-S3 uses Amazon S3 managed keys, while SSE-KMS uses AWS KMS keys, providing key management, auditing, and fine-grained access control.

---

## Question 27

What is Envelope Encryption?

### Answer

Envelope Encryption encrypts the data using a data key, and then encrypts that data key using a KMS key.

---

## Question 28

Why should HTTPS always be used with Amazon S3?

### Answer

HTTPS encrypts data in transit, protecting it from interception while it travels between the client and Amazon S3.

---

## Question 29

What are S3 Bucket Keys?

### Answer

S3 Bucket Keys reduce the number of AWS KMS requests for server-side encryption with KMS, lowering KMS costs for large-scale workloads.

---

## Question 30

When should SSE-KMS be preferred over SSE-S3?

### Answer

Use SSE-KMS when you require centralized key management, audit logging, fine-grained access control, or regulatory compliance.

---

# 109. Hands-on Labs

## Lab 16

Create a bucket with default SSE-S3 encryption.

---

## Lab 17

Create a bucket with default SSE-KMS encryption.

---

## Lab 18

Upload objects using different encryption methods and compare their metadata.

---

## Lab 19

Create a Customer Managed KMS Key and configure an S3 bucket to use it.

---

## Lab 20

Enable S3 Bucket Keys and observe the reduction in KMS request usage (in a suitable workload).

---

# 110. One-Page Revision

```
Upload File

↓

HTTPS

↓

Amazon S3

↓

Encryption

↓

SSE-S3

OR

SSE-KMS

OR

SSE-C

↓

Encrypted Object
```

Remember:

- Encryption at Rest protects stored data.
- Encryption in Transit protects network traffic.
- SSE-S3 → AWS manages keys.
- SSE-KMS → AWS KMS manages keys with customer control.
- SSE-C → Customer supplies the key.
- Client-Side Encryption → Client encrypts before upload.
- Bucket Keys reduce KMS costs.
- Enable Default Bucket Encryption.

---

# Think Like a Production Engineer

Encryption should never be optional for production workloads.

When designing secure S3 storage:

1. Enable default encryption on every bucket.
2. Use SSE-KMS with customer-managed keys for sensitive workloads.
3. Restrict KMS key usage using IAM and key policies.
4. Enforce HTTPS using bucket policies.
5. Enable CloudTrail to audit KMS operations.
6. Use Bucket Keys to optimize KMS costs at scale.
7. Review encryption settings during security audits.

A production-grade S3 environment is **encrypted by default, audited continuously, and protected by least-privilege access controls**.

# End of Part 4
