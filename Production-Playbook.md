# AWS DevOps Playbook

# Production Playbook

---

# Purpose

This playbook is a quick production troubleshooting guide covering the topics
studied across all volumes.

Use this before interviews, during on-call incidents, and while debugging
production environments.

---

# Universal Debugging Flow

```
Alert

↓

Identify Scope

↓

Collect Evidence

↓

Check Recent Changes

↓

Infrastructure

↓

Application

↓

Logs

↓

Fix

↓

Validation
```

Never start changing configurations without identifying the root cause.

---

# EC2

## EC2 Not Reachable

Check

- Instance State
- Security Group
- NACL
- Route Table
- Internet Gateway
- SSH Key
- EC2 Status Checks
- CloudWatch Metrics
- Disk Full

Commands

```bash
ping

ssh

systemctl status

journalctl -xe

df -h

free -m

top

htop
```

---

# High CPU

Check

CloudWatch

↓

top

↓

htop

↓

Application

↓

Auto Scaling

---

# High Memory

Check

```
free -m

top

ps aux --sort=-%mem
```

---

# EBS

Problems

- Volume not attached
- Volume full
- Filesystem corruption

Commands

```bash
lsblk

df -h

mount

blkid
```

---

# AMI

Verify

- Correct region
- Correct architecture
- Latest security patches
- Required software included

---

# Auto Scaling

Common Problems

- Scaling policy missing
- Health check failure
- Launch Template issue
- Instance warm-up
- Capacity limits

---

# ALB

502

Check

- Target Group
- Container Port
- Health Check
- Application Running

503

Check

- Healthy Targets
- Readiness Probe
- Service Endpoints

404

Check

- Listener Rules
- Path Routing

---

# Docker

Container Exits

Check

```bash
docker logs

docker inspect

docker ps -a
```

---

# Docker Networking

Container cannot communicate

Check

- Network
- DNS
- Port Mapping
- Firewall

---

# Docker Volumes

Data Missing

Check

```bash
docker volume ls

docker inspect
```

---

# Docker Compose

Check

```bash
docker compose ps

docker compose logs
```

---

# Kubernetes / EKS

Pod Pending

↓

describe pod

CrashLoopBackOff

↓

logs

ImagePullBackOff

↓

Image

Registry

Secrets

Node NotReady

↓

kubelet

EC2

Disk

Ingress Failure

↓

ALB

Controller

IRSA

DNS Failure

↓

CoreDNS

Service

PVC Pending

↓

StorageClass

CSI Driver

IRSA Failure

↓

Service Account

IAM

Trust Policy

---

# CloudWatch

Always verify

- CPU
- Memory
- Disk
- Network
- Logs
- Alarms
- Events

---

# CloudTrail

Use CloudTrail for

- Who deleted EC2?
- Who modified IAM?
- Who changed Security Groups?
- Who deleted S3 bucket?
- API audit trail

---

# General Production Checklist

✓ Infrastructure

✓ Network

✓ Storage

✓ IAM

✓ Logs

✓ Monitoring

✓ Security

✓ Recent Deployment

✓ Application

---

# Golden Rule

Never guess.

Collect evidence.

Find the root cause.

Then fix.
