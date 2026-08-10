# Amazon Elastic Kubernetes Service (EKS)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 08
>
> Part 5 – Production Troubleshooting Playbook

---

# Chapter Objective

After completing this chapter, you should be able to:

- Troubleshoot production EKS clusters
- Follow a structured debugging process
- Identify root causes quickly
- Handle networking, storage, IAM and scheduling issues
- Answer Amazon-style troubleshooting interviews

---

# Golden Rule of Troubleshooting

Never start by changing configurations.

Always collect evidence first.

Follow this order:

```
User Complaint
      │
What changed?
      │
kubectl get
      │
kubectl describe
      │
Logs
      │
Events
      │
AWS Components
      │
Root Cause
      │
Fix
      │
Validation
```

---

# Universal Debugging Flow

```
Problem

↓

Pods

↓

Deployment

↓

ReplicaSet

↓

Node

↓

Service

↓

Ingress

↓

ALB

↓

IAM

↓

Storage

↓

Network

↓

Application
```

---

# Scenario 1

## Pod Pending

Symptoms

```
STATUS

Pending
```

Investigation

```bash
kubectl describe pod <pod>
```

Possible Reasons

- Insufficient CPU
- Insufficient Memory
- No Pod IPs
- PVC Pending
- Taints
- Affinity rules
- NodeSelector mismatch

Fix

Depends on the event message.

Never assume Cluster Autoscaler is the answer.

---

# Scenario 2

## CrashLoopBackOff

Meaning

The Pod starts but repeatedly crashes.

Common Causes

- Application exception
- Wrong environment variable
- Missing Secret
- Database connection failure
- Invalid command
- Port conflict

Commands

```bash
kubectl logs pod-name

kubectl logs pod-name --previous
```

Interview Tip

CrashLoopBackOff is an application problem, not a scheduler problem.

---

# Scenario 3

## ImagePullBackOff

Meaning

Image cannot be downloaded.

Check

- Image name
- Image tag
- ECR repository
- IRSA/Execution permissions
- Registry authentication
- Network connectivity

Commands

```bash
kubectl describe pod
```

---

# Scenario 4

## ErrImagePull

Usually occurs before ImagePullBackOff.

Possible Causes

- Wrong repository
- Wrong image tag
- Private registry authentication failure

---

# Scenario 5

## Node NotReady

Commands

```bash
kubectl get nodes

kubectl describe node
```

Check

- kubelet
- EC2 health
- Disk pressure
- Memory pressure
- Network
- IAM

---

# Scenario 6

## ALB Not Created

Check

```bash
kubectl get ingress

kubectl describe ingress
```

Verify

- AWS Load Balancer Controller
- IRSA
- IAM
- Subnet tags
- Controller logs

---

# Scenario 7

## ALB Returns 502

Possible Causes

- Wrong target port
- Container port mismatch
- Application crash
- Health check path
- Target group unhealthy

---

# Scenario 8

## ALB Returns 503

Meaning

No healthy backend targets.

Check

- Target Group
- Pod readiness
- Service endpoints
- Deployment

---

# Scenario 9

## Service Not Reachable

Commands

```bash
kubectl get svc

kubectl get endpoints
```

Verify

- Labels
- Selectors
- Service type
- kube-proxy

---

# Scenario 10

## DNS Failure

Symptoms

```
backend.default.svc.cluster.local

Cannot resolve
```

Check

```bash
kubectl get pods -n kube-system

kubectl logs deployment/coredns -n kube-system
```

---

# Scenario 11

## CoreDNS Crash

Possible Causes

- Wrong ConfigMap
- Resource exhaustion
- Networking issue

---

# Scenario 12

## Pod Cannot Access S3

Check

- Service Account
- IRSA annotation
- IAM Role
- Trust Policy
- Bucket Policy

---

# Scenario 13

## AccessDenied

Never guess.

Determine

```
IAM?

RBAC?

Bucket Policy?

KMS?

Resource Policy?
```

---

# Scenario 14

## Secret Not Mounted

Verify

```bash
kubectl describe pod
```

Check

- Secret exists
- Namespace
- Mount path
- IAM permissions

---

# Scenario 15

## PVC Pending

Commands

```bash
kubectl get pvc

kubectl describe pvc
```

Possible Causes

- StorageClass
- No PV
- CSI Driver
- EBS limits

---

# Scenario 16

## EBS Volume Not Attached

Check

- CSI Controller
- IAM
- Availability Zone
- StorageClass

---

# Scenario 17

## Cluster Autoscaler Not Working

Check

- Pending reason
- ASG tags
- IAM permissions
- Cluster Autoscaler logs

Remember

Cluster Autoscaler only solves resource shortages.

---

# Scenario 18

## Karpenter Not Creating Nodes

Verify

- IAM
- EC2 quota
- Provisioner/NodePool configuration
- Subnet discovery
- Security Groups

---

# Scenario 19

## High CPU

Check

CloudWatch

↓

kubectl top

↓

Application

↓

HPA

↓

Scale

---

# Scenario 20

## High Memory

Check

```bash
kubectl top pod
```

Possible Causes

- Memory leak
- Incorrect limits
- OOMKilled

---

# Scenario 21

## OOMKilled

Exit Code

```
137
```

Root Cause

Container exceeded memory limit.

Fix

- Increase limit
- Optimize application

---

# Scenario 22

## Pod Scheduled on Wrong Node

Check

- Node Selector
- Node Affinity
- Taints
- Tolerations

---

# Scenario 23

## Pods Never Scale

Check

```bash
kubectl get hpa
```

Verify

- Metrics Server
- Resource requests
- HPA events

---

# Scenario 24

## Metrics Server Missing

Symptoms

```bash
kubectl top pod
```

returns

```
Metrics API not available
```

Install Metrics Server.

---

# Scenario 25

## Worker Node Full

Symptoms

- DiskPressure
- MemoryPressure

Commands

```bash
kubectl describe node
```

---

# Scenario 26

## Container Cannot Reach Database

Check

- Security Groups
- Route Tables
- DNS
- Database endpoint
- Network Policy
- Port

---

# Scenario 27

## Pod Restarting Frequently

Commands

```bash
kubectl get pod

kubectl describe pod

kubectl logs
```

---

# Scenario 28

## Namespace Deleted Accidentally

Check

- GitOps repository
- Backup
- Velero restore
- IaC manifests

---

# Scenario 29

## API Server Slow

Possible Causes

- Large cluster
- Excessive API requests
- Control plane issue

---

# Scenario 30

## Entire Cluster Down

Investigation Order

```
AWS Health

↓

EKS Status

↓

Control Plane

↓

Worker Nodes

↓

Networking

↓

Application
```

---

# Production Checklist

Always Verify

- Pods
- Deployments
- ReplicaSets
- Nodes
- Services
- Endpoints
- Ingress
- ALB
- CoreDNS
- kube-proxy
- IRSA
- IAM
- PVC
- StorageClass
- CSI Driver
- CloudWatch
- Events
- Logs

---

# Common Error Mapping

| Error | First Place to Check |
|--------|----------------------|
| Pending | `kubectl describe pod` |
| CrashLoopBackOff | Application logs |
| ImagePullBackOff | Image + ECR |
| ErrImagePull | Registry |
| OOMKilled | Memory |
| Node NotReady | kubelet + EC2 |
| ALB 502 | Target Group |
| ALB 503 | Healthy Targets |
| DNS Failure | CoreDNS |
| AccessDenied | IRSA / IAM |
| Secret Missing | Secret + Mount |
| PVC Pending | StorageClass |
| HPA Not Scaling | Metrics Server |
| Cluster Autoscaler | Pending reason |
| Pod Restarting | Logs |

---

# Amazon Interview Questions

## Question 1

A Pod is Pending. What is your first step?

### Perfect Answer

I run:

```bash
kubectl describe pod <pod-name>
```

to inspect scheduling events. The event messages identify whether the issue is CPU, memory, taints, affinity, PVCs, IP exhaustion, or another scheduling constraint.

---

## Question 2

CrashLoopBackOff vs ImagePullBackOff?

### Perfect Answer

CrashLoopBackOff means the container starts and then repeatedly crashes. ImagePullBackOff means Kubernetes cannot download the container image, so the container never starts.

---

## Question 3

Your ALB exists but returns 503.

### Perfect Answer

I verify Target Group health, Service endpoints, Pod readiness, Deployment status, and health check configuration to identify why there are no healthy backend targets.

---

## Question 4

Application gets AccessDenied.

### Perfect Answer

I determine whether the failure is related to IAM, IRSA, RBAC, bucket policies, or KMS permissions before making any changes.

---

## Question 5

Cluster Autoscaler isn't adding nodes.

### Perfect Answer

I first confirm the Pods are pending due to insufficient resources. Then I verify the Cluster Autoscaler logs, IAM permissions, Auto Scaling Group tags, and node group configuration.

---

## Amazon Cross Questions

### Question

A Pod is Pending, but CPU and memory are available. What else could be wrong?

### Perfect Answer

I would investigate taints, tolerations, node affinity, node selectors, PVC binding, Pod IP exhaustion, and scheduling constraints because Pending is not always caused by insufficient resources.

---

### Question

How do you differentiate an application issue from an infrastructure issue?

### Perfect Answer

I begin by checking Kubernetes events and infrastructure health. If the infrastructure is healthy and the container is running, I review application logs and dependencies such as databases or external APIs to isolate application-level failures.

---

### Question

A deployment succeeded, but users still see errors.

### Perfect Answer

I verify Pod readiness, Service endpoints, ALB Target Group health, application logs, and dependent services such as databases before concluding the deployment itself is faulty.

---

# Hands-on Labs (To Perform Later)

## Lab 1

Trigger CrashLoopBackOff by using an invalid command.

---

## Lab 2

Trigger ImagePullBackOff using an incorrect image tag.

---

## Lab 3

Create a Pending Pod with excessive CPU requests.

---

## Lab 4

Break a Service selector and troubleshoot connectivity.

---

## Lab 5

Deploy an Ingress with an incorrect health check path and observe ALB failures.

---

## Lab 6

Create a Pod without the required IRSA permissions and observe `AccessDenied`.

---

## Lab 7

Create an oversized PVC request and investigate why it remains Pending.

---

# One-Page Debugging Flow

```
Problem
   │
kubectl get
   │
kubectl describe
   │
Events
   │
Logs
   │
AWS Resources
   │
Root Cause
   │
Fix
   │
Validate
```

---

# Think Like a Production Engineer

Don't memorize commands.

Memorize **where to look first**.

For every incident, ask:

1. Is it a scheduling issue?
2. Is it a networking issue?
3. Is it an IAM issue?
4. Is it a storage issue?
5. Is it an application issue?

Always narrow the problem layer by layer instead of changing multiple things at once.

# End of Part 5
