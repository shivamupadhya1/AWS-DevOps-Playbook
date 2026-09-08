# AWS EC2 Instance Access Methods — Interview & Hands-on Notes

## Overview

AWS provides multiple ways to access an EC2 instance:

| Method | Typical Use | SSH Key | Inbound Port 22 | Public IP | IAM Authentication |
|---|---|---:|---:|---:|---:|
| **EC2 Instance Connect** | Public/reachable EC2 | Temporary | Usually required | Usually | Yes, for authorization |
| **EC2 Instance Connect Endpoint** | Private EC2 | Temporary | Not necessarily | No | Yes |
| **SSM Session Manager** | Secure production access | No | No | No | Yes |
| **EC2 Serial Console** | Emergency/low-level troubleshooting | No SSH | No | No | AWS console/IAM access |

### Easy way to remember

```text
EC2 Instance Connect
        ↓
      SSH
        ↓
Temporary SSH key

EC2 Instance Connect Endpoint
        ↓
      SSH
        ↓
Private EC2 connectivity

SSM Session Manager
        ↓
      SSM
        ↓
IAM-based access
        ↓
No inbound SSH required

EC2 Serial Console
        ↓
   Serial port
        ↓
Boot / OS / network troubleshooting
```

---

# 1. EC2 Instance Connect

## What is it?

EC2 Instance Connect is an AWS service that allows you to connect to an EC2 instance using **temporary SSH keys** instead of managing a permanent SSH key for every connection.

It is still an **SSH-based connection**.

### Important interview point

> EC2 Instance Connect does NOT eliminate SSH. It simplifies SSH key management by using temporary public keys.

---

## Normal SSH vs EC2 Instance Connect

### Traditional SSH

```bash
ssh -i mykey.pem ec2-user@PUBLIC_IP
```

You normally need:

- Private SSH key
- Username
- Public IP/DNS
- Network connectivity
- Security Group allowing TCP/22
- SSH service running on the instance

### EC2 Instance Connect

Conceptually:

```text
Your Laptop
     |
     | AWS authentication
     v
EC2 Instance Connect
     |
     | Temporary public key
     v
EC2 Instance
     |
     | SSH
     v
sshd
```

AWS pushes a temporary public key to the instance and the SSH connection uses it.

---

## Requirements

For a typical public EC2 instance:

### 1. Network connectivity

Your client must be able to reach the instance.

Typical architecture:

```text
Internet
   |
   v
Internet Gateway
   |
   v
Public Subnet
   |
   v
EC2 Instance
```

### 2. Security Group

TCP port 22 generally needs to be allowed.

Example:

```text
Inbound rule:

Type: SSH
Protocol: TCP
Port: 22
Source: Your-IP/32
```

Avoid unnecessarily allowing:

```text
0.0.0.0/0
```

### 3. EC2 Instance Connect support

The instance needs the appropriate EC2 Instance Connect support/package.

---

# Interview Questions — EC2 Instance Connect

## Q1. What is EC2 Instance Connect?

### Answer

> EC2 Instance Connect is an AWS service that provides temporary SSH-key-based access to EC2 instances. AWS pushes a temporary public key to the instance, reducing the need to manage long-lived SSH keys.

---

## Q2. Does EC2 Instance Connect use SSH?

### Answer

> Yes. EC2 Instance Connect ultimately uses SSH for the connection.

---

## Q3. Does EC2 Instance Connect eliminate port 22?

### Answer

> No. Since it uses SSH, the instance still needs a network path for SSH and TCP port 22 must generally be reachable.

---

## Q4. Why use EC2 Instance Connect instead of a permanent `.pem` key?

### Answer

> It reduces the need to distribute and manage long-lived SSH private keys. Access can be controlled using IAM, while the SSH public key used for the connection is temporary.

---

# 2. EC2 Instance Connect Endpoint

## What problem does it solve?

Suppose your EC2 instance is in a private subnet:

```text
VPC
|
+-- Public Subnet
|
+-- Private Subnet
       |
       +-- EC2
           10.0.2.10
```

The instance has:

- No public IP
- No direct Internet SSH access

You cannot simply do:

```bash
ssh ec2-user@10.0.2.10
```

from your laptop over the Internet.

Traditionally, you could use a bastion host:

```text
Laptop
   |
   v
Bastion Host
   |
   v
Private EC2
```

EC2 Instance Connect Endpoint provides another approach.

---

## Architecture

```text
Your Laptop
     |
     | Authenticated connection
     v
EC2 Instance Connect Endpoint
     |
     | VPC private connectivity
     v
Private EC2
```

The EC2 instance does not need a public IP.

---

## Very Important

EC2 Instance Connect Endpoint is **not SSM**.

It is still associated with SSH-style access.

Remember:

```text
EC2 Instance Connect
        ↓
      SSH

EC2 Instance Connect Endpoint
        ↓
      SSH

SSM Session Manager
        ↓
      SSM Agent
        ↓
IAM
```

---

# Interview Questions — EC2 Instance Connect Endpoint

## Q1. What is EC2 Instance Connect Endpoint?

### Answer

> EC2 Instance Connect Endpoint provides a way to use EC2 Instance Connect to reach EC2 instances in private subnets without requiring the instances to have public IP addresses or requiring a bastion host.

---

## Q2. How would you connect to a private EC2 instance?

### Answer

> I have several options depending on the architecture. I could use a bastion host, VPN, Direct Connect, SSM Session Manager, or EC2 Instance Connect Endpoint. If the requirement is specifically SSH access without a public IP, EC2 Instance Connect Endpoint is an AWS-native option.

---

## Q3. Difference between EC2 Instance Connect and EC2 Instance Connect Endpoint?

### Answer

> EC2 Instance Connect is commonly used for reachable instances, including public instances. EC2 Instance Connect Endpoint provides private VPC connectivity so EC2 Instance Connect can be used with instances that don't have public IP addresses.

---

# 3. SSM Session Manager

## What is it?

AWS Systems Manager Session Manager provides shell access to EC2 instances using **IAM authentication and the SSM Agent**.

It is often preferred for production administration because it can eliminate:

- SSH keys
- Bastion hosts
- Public IPs
- Inbound SSH port 22

---

## Architecture

```text
                 IAM
                  |
                  v
Your Laptop ---> SSM Session Manager
                  |
                  v
              SSM Agent
                  |
                  v
                 EC2
```

---

## Key difference from SSH

Traditional SSH:

```text
Client
  |
  | TCP 22
  v
EC2
```

SSM:

```text
Administrator
      |
      | IAM authentication
      v
Systems Manager
      |
      v
SSM Agent
      |
      v
EC2
```

There is no requirement to expose inbound SSH port 22.

---

# SSM Requirements

## 1. SSM Agent

The EC2 instance needs the SSM Agent.

Many AWS-provided AMIs already include it.

---

## 2. IAM Role

The EC2 instance needs an appropriate IAM role for Systems Manager.

A common AWS-managed policy is:

```text
AmazonSSMManagedInstanceCore
```

The instance role gives the SSM Agent permission to communicate with Systems Manager.

---

## 3. Network Connectivity

This is an important interview point.

SSM does NOT mean the instance needs zero networking.

The instance needs a way to communicate with AWS Systems Manager.

For private subnets, this can be achieved using appropriate VPC endpoints.

Common Systems Manager-related endpoints include:

```text
ssm
ssmmessages
ec2messages
```

The exact endpoint requirements can vary by Region and current Systems Manager architecture.

### Key interview statement

> SSM does not require inbound SSH connectivity, but the instance still needs outbound connectivity to the Systems Manager service.

---

# Starting an SSM Session

AWS CLI example:

```bash
aws ssm start-session --target i-xxxxxxxxxxxxxxxxx
```

You can also initiate sessions from the AWS Console.

---

# Interview Questions — SSM

## Q1. Why use SSM Session Manager instead of SSH?

### Answer

> For production environments, I prefer SSM because it provides IAM-based access without distributing SSH keys, exposing inbound port 22, or maintaining bastion hosts. It also provides centralized access control and session logging capabilities.

---

## Q2. Can SSM connect to a private EC2 instance?

### Answer

> Yes. A private EC2 instance can be accessed using SSM without a public IP. The instance needs the SSM Agent, an appropriate IAM role, and network connectivity to Systems Manager, potentially through VPC endpoints.

---

## Q3. Does SSM require port 22?

### Answer

> No. SSM Session Manager does not require inbound SSH port 22.

---

## Q4. Does SSM require a public IP?

### Answer

> No. A private instance can use SSM as long as it has the required IAM permissions and connectivity to Systems Manager.

---

## Q5. SSM is not showing my EC2 instance. What do you check?

### Answer

I would check:

1. Is the EC2 instance running?
2. Is SSM Agent installed?
3. Is SSM Agent running?
4. Does the instance have the correct IAM role?
5. Does the instance have network connectivity to Systems Manager?
6. If private, are the required VPC endpoints configured correctly?
7. Are DNS resolution and routing working?
8. Are there OS-level problems preventing the agent from communicating?

---

# 4. EC2 Serial Console

## What is it?

EC2 Serial Console provides access to the instance's serial port for low-level troubleshooting.

It is especially useful when normal network-based access isn't working.

---

## Typical scenario

Imagine:

```text
EC2
 |
 +-- SSH broken
 +-- Network broken
 +-- Firewall problem
 +-- Routing problem
 +-- Boot problem
```

You can't SSH.

SSM might also be unavailable.

Serial Console can provide another troubleshooting path.

---

## Architecture

```text
AWS Console
     |
     v
EC2 Serial Console
     |
     v
EC2 Serial Port
     |
     v
Operating System
```

---

## Important characteristic

Serial Console does not depend on normal instance network connectivity.

Therefore:

```text
SSH              ❌
Network access   ❌
SSM              ❌
Serial Console   potentially available
```

It is designed for troubleshooting certain boot, OS, and network issues.

---

# What can you troubleshoot?

Examples:

### Boot issues

```text
Kernel problems
Boot failures
GRUB-related issues
```

### Network issues

```text
Incorrect network configuration
Routing problems
Firewall configuration
```

### OS-level problems

```text
SSH service problems
OS configuration issues
Emergency troubleshooting
```

---

# Interview Questions — Serial Console

## Q1. What is EC2 Serial Console?

### Answer

> EC2 Serial Console provides access to an EC2 instance's serial port for low-level troubleshooting. It can be useful for boot, OS, and network problems when normal network-based access such as SSH is unavailable.

---

## Q2. Does Serial Console require network connectivity to the EC2 instance?

### Answer

> It does not depend on the instance's normal network path, which makes it useful when network connectivity is broken.

---

## Q3. When would you use Serial Console?

### Answer

> I would use it for emergency troubleshooting when SSH and potentially SSM are unavailable, especially for boot, OS, or network-level issues.

---

# 5. Complete Comparison

| Feature | Instance Connect | Instance Connect Endpoint | SSM Session Manager | Serial Console |
|---|---|---|---|---|
| SSH-based | Yes | Yes | No | No |
| Temporary SSH key | Yes | Yes | No | No |
| IAM-based authorization | Yes | Yes | Yes | Yes |
| Public IP required | Typically for public access | No | No | No |
| Inbound port 22 | Usually | Not necessarily | No | No |
| Bastion required | No | No | No | No |
| SSM Agent required | No | No | Yes | No |
| Normal network path required | Yes | VPC connectivity | Agent-to-AWS connectivity | No |
| Best use | Quick SSH access | Private SSH access | Secure administration | Emergency troubleshooting |

---

# 6. Production Security Perspective

## Traditional SSH

```text
Internet
   |
   | TCP 22
   v
EC2
```

Potentially larger attack surface.

---

## EC2 Instance Connect

```text
IAM
 |
 v
Temporary SSH Key
 |
 v
TCP 22
 |
 v
EC2
```

Better key-management model, but it is still SSH/network based.

---

## EC2 Instance Connect Endpoint

```text
IAM
 |
 v
EIC Endpoint
 |
 v
Private EC2
```

Allows SSH-style access without exposing the instance through a public IP.

---

## SSM Session Manager

```text
IAM
 |
 v
SSM
 |
 v
SSM Agent
 |
 v
Private EC2
```

No inbound SSH is required.

---

## Serial Console

```text
AWS Console
     |
     v
Serial Interface
     |
     v
EC2
```

Primarily an emergency/troubleshooting mechanism.

---

# 7. Scenario-Based Interview Questions

## Scenario 1

### Interviewer

> You have a production EC2 instance in a private subnet. It has no public IP and port 22 is not open. How will you connect?

### Strong answer

> I would prefer SSM Session Manager if the instance is managed by Systems Manager. It doesn't require a public IP, inbound SSH, or a bastion host. I would verify the SSM Agent, IAM role, and connectivity to Systems Manager.

---

# Scenario 2

### Interviewer

> The company doesn't want port 22 open on production servers. How will administrators access them?

### Answer

> I would use SSM Session Manager with IAM-based authorization. This avoids exposing inbound SSH port 22 and eliminates the need to distribute SSH keys.

---

# Scenario 3

### Interviewer

> The EC2 instance is private, but the team specifically requires SSH access. What AWS service can you use?

### Answer

> EC2 Instance Connect Endpoint can provide a way to connect to the private instance without giving it a public IP or requiring a bastion host.

---

# Scenario 4

### Interviewer

> SSH is not working and SSM is also unavailable. What can you use for deeper troubleshooting?

### Answer

> I would consider EC2 Serial Console, assuming it is supported for the instance and enabled for the account. It can provide low-level access for troubleshooting boot, OS, or network problems.

---

# Scenario 5

### Interviewer

> SSH service has stopped on the EC2 instance. Can EC2 Instance Connect solve the problem?

### Answer

> No. EC2 Instance Connect ultimately uses SSH, so if the SSH daemon is unavailable, Instance Connect won't solve that problem. I would use SSM if it is available, or Serial Console for deeper troubleshooting.

---

# Scenario 6

### Interviewer

> Can SSM connect to an EC2 instance with no public IP?

### Answer

> Yes. SSM can work with private instances. The instance needs the SSM Agent, the appropriate IAM role, and connectivity to Systems Manager, which can be provided through appropriate VPC endpoints in a private environment.

---

# Scenario 7

### Interviewer

> Does SSM mean the EC2 instance doesn't need networking?

### Answer

> No. The instance still needs connectivity to the Systems Manager service. The important difference is that I don't need inbound connectivity from my laptop to the EC2 instance.

---

# Scenario 8

### Interviewer

> What is the main security advantage of SSM over SSH?

### Answer

> SSM can use IAM-based authentication and does not require inbound SSH port 22 or long-lived SSH private keys. This reduces the attack surface and simplifies centralized access control.

---

# 8. Troubleshooting Cheat Sheet

## EC2 Instance Connect not working

Check:

```text
1. Instance running?
2. Correct username?
3. Public/reachable network path?
4. Security Group allows TCP 22?
5. NACL/routing correct?
6. SSH service running?
7. EC2 Instance Connect support/package present?
8. IAM permissions correct?
```

---

## EC2 Instance Connect Endpoint not working

Check:

```text
1. Endpoint exists?
2. Endpoint is correctly associated with the VPC?
3. Target instance is reachable within the VPC?
4. Security Groups correct?
5. Network ACLs correct?
6. Route tables correct?
7. IAM permissions correct?
8. SSH service running?
9. Instance Connect support/package present?
```

---

## SSM not working

Check:

```text
1. EC2 running?
2. SSM Agent installed?
3. SSM Agent running?
4. Correct IAM instance role?
5. AmazonSSMManagedInstanceCore or equivalent permissions?
6. Outbound network connectivity?
7. Private subnet VPC endpoints if needed?
8. DNS resolution?
9. Security Groups/NACLs?
10. OS/agent logs?
```

---

## Serial Console not working

Check:

```text
1. Is Serial Console supported for the instance?
2. Is account-level access enabled?
3. Does the IAM identity have required permissions?
4. Is the instance state appropriate?
5. Is the OS configured with a usable serial console?
```

---

# 9. Interviewer's Favorite Comparison

### Question

> EC2 Instance Connect vs EC2 Instance Connect Endpoint vs SSM — explain in one answer.

### Best answer

> "EC2 Instance Connect is temporary-key-based SSH access to a reachable EC2 instance. EC2 Instance Connect Endpoint extends this capability to private instances without requiring a public IP or bastion host. SSM Session Manager is different because it uses the SSM Agent and IAM-based authentication rather than SSH, and it doesn't require inbound port 22. For production administration, I would generally prefer SSM because it reduces SSH exposure and simplifies access management."

---

# 10. One-Minute Interview Answer

If the interviewer shows the AWS connection screen and says:

> "Explain these four options."

Say:

> "There are four different access mechanisms. EC2 Instance Connect provides temporary SSH-key-based access to a reachable EC2 instance. EC2 Instance Connect Endpoint provides a private VPC path so I can use Instance Connect with private instances without a public IP or bastion host. SSM Session Manager is IAM-based and is generally my preferred production option because it doesn't require SSH keys, inbound port 22, public IPs, or bastion hosts. Finally, EC2 Serial Console provides low-level serial access for troubleshooting boot, OS, or network problems when normal network-based access isn't working."

---

# 11. Memory Trick

Remember:

```text
                    EC2 ACCESS
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
     SSH              SSH              SSM
       |                |                |
   Instance           EIC EP            IAM
   Connect             |                |
       |            Private EC2          |
   Reachable                            EC2
       |
       |
       +----------------------------------+

             Emergency / Troubleshooting
                        |
                        v
                  Serial Console
```

### Short version

```text
Instance Connect
= Temporary SSH

Instance Connect Endpoint
= Temporary SSH + Private EC2

SSM
= IAM + Agent + No inbound SSH

Serial Console
= Emergency low-level access
```

---

# 12. Hands-on Lab Plan

For interview preparation, build these labs instead of only memorizing theory.

## Lab 1 — Public EC2 + Instance Connect

Create:

```text
VPC
 |
Public Subnet
 |
EC2
 |
Public IP
 |
TCP 22
```

Practice:

- Connect using EC2 Instance Connect
- Check SSH
- Check Security Group
- Understand temporary key behavior

---

## Lab 2 — Private EC2 + Instance Connect Endpoint

Create:

```text
VPC
 |
Private Subnet
 |
EC2
 |
No Public IP

EC2 Instance Connect Endpoint
 |
 +----> Private EC2
```

Practice:

- Connect to private EC2
- Verify no public IP
- Understand endpoint networking
- Troubleshoot Security Groups/routes

---

## Lab 3 — Private EC2 + SSM

Create:

```text
Private EC2
    |
    +-- SSM Agent
    |
    +-- IAM Role
    |
    +-- Network connectivity
             |
             v
        AWS Systems Manager
```

Practice:

- Start an SSM session
- Verify no port 22 is required
- Remove/modify access permissions and troubleshoot
- Test private connectivity

---

## Lab 4 — Serial Console

Practice:

```text
EC2
 |
Break/inspect network or OS configuration
 |
SSH unavailable
 |
Serial Console
 |
Troubleshoot/recover
```

---

# Final Interview Checklist

You should be able to answer these without notes:

```text
[ ] What is EC2 Instance Connect?
[ ] Does Instance Connect use SSH?
[ ] Does Instance Connect require port 22?
[ ] What is EC2 Instance Connect Endpoint?
[ ] Why use Endpoint for a private EC2?
[ ] Instance Connect vs Instance Connect Endpoint?
[ ] What is SSM Session Manager?
[ ] Why is SSM preferred in production?
[ ] Does SSM require port 22?
[ ] Does SSM require a public IP?
[ ] What is SSM Agent?
[ ] What IAM role does SSM need?
[ ] How does SSM work with private subnets?
[ ] What are VPC endpoints?
[ ] What is EC2 Serial Console?
[ ] When would you use Serial Console?
[ ] Can Serial Console work when networking is broken?
[ ] SSH broken — what do you check?
[ ] SSM unavailable — what do you check?
[ ] Private EC2 — how do you connect?
[ ] No port 22 allowed — which method?
[ ] SSH and SSM both unavailable — what next?
```

## The key decision tree

```text
Need EC2 access?
       |
       +-- Public/reachable + SSH needed
       |          |
       |          +--> EC2 Instance Connect
       |
       +-- Private + SSH specifically required
       |          |
       |          +--> EC2 Instance Connect Endpoint
       |
       +-- Secure production administration
       |          |
       |          +--> SSM Session Manager
       |
       +-- Network/boot/OS emergency
                  |
                  +--> EC2 Serial Console
```

