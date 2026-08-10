# Amazon Elastic Kubernetes Service (EKS)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 08
>
> Part 3 – Networking (VPC CNI, Services, DNS & Ingress)

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain EKS networking architecture
- Understand AWS VPC CNI
- Explain how Pods receive IP addresses
- Understand ENIs
- Explain CoreDNS
- Explain kube-proxy
- Understand Kubernetes Services
- Understand ALB & NLB integration
- Troubleshoot networking issues
- Answer networking interview questions confidently

---

# 1. Networking Architecture

```
Internet

↓

Application Load Balancer

↓

Ingress

↓

Kubernetes Service

↓

Pod

↓

Application
```

This is the path followed by most production HTTP requests.

---

# 2. Pod Networking

⭐⭐⭐⭐⭐

Every Pod gets its own IP address.

Unlike Docker bridge networking, Pods do not share a single host IP.

Example

```
Node

↓

Pod A

10.0.1.21

Pod B

10.0.1.22
```

Pods communicate directly using IP.

---

# 3. AWS VPC CNI

⭐⭐⭐⭐⭐

AWS VPC CNI is the networking plugin used by EKS.

Responsibilities

- Assign Pod IP addresses
- Attach ENIs
- Configure routing
- Connect Pods directly to the VPC

Without the VPC CNI, Pods cannot receive VPC IP addresses.

---

# 4. Pod IP Allocation

Flow

```
Node Starts

↓

Primary ENI Attached

↓

Secondary IPs Allocated

↓

Pod Created

↓

Pod Gets One Secondary IP
```

Each Pod receives an IP from the node's available secondary IP pool.

---

# 5. Elastic Network Interface (ENI)

⭐⭐⭐⭐⭐

Each EC2 worker node has one or more ENIs.

Example

```
EC2

↓

ENI

↓

Secondary IPs

↓

Pods
```

Pods use secondary private IPs from the ENI.

---

# 6. Why Pods Sometimes Stay Pending

If the node has exhausted available secondary IP addresses:

```
No Free IP

↓

Pod Pending
```

Adding CPU or memory will not solve this.

Solutions

- Increase ENI/IP capacity
- Use larger instance types
- Add more worker nodes
- Enable Prefix Delegation where appropriate

---

# 7. Prefix Delegation

Instead of allocating individual secondary IPs,

AWS can assign prefixes to improve Pod density.

Benefits

- Faster Pod startup
- More Pods per node
- Better scalability

---

# 8. Kubernetes Service

Pods are temporary.

Their IP addresses change.

A Service provides a stable endpoint.

```
Client

↓

Service

↓

Pod
```

Applications should communicate using Services rather than Pod IPs.

---

# 9. Types of Services

```
ClusterIP

NodePort

LoadBalancer

ExternalName
```

---

# 10. ClusterIP

Default Service type.

Accessible only inside the cluster.

Example

```
Frontend

↓

Backend Service

↓

Backend Pods
```

---

# 11. NodePort

Exposes the application on every worker node.

Example

```
Node IP

↓

Port 30080

↓

Pod
```

Useful for testing but rarely used directly in production.

---

# 12. LoadBalancer

Creates an AWS Load Balancer.

```
Internet

↓

AWS Load Balancer

↓

Service

↓

Pods
```

Suitable for external applications.

---

# 13. ExternalName

Maps a Kubernetes Service to an external DNS name.

Example

```
database.company.com
```

Useful when integrating external services.

---

# 14. kube-proxy

⭐⭐⭐⭐⭐

Runs on every worker node.

Responsibilities

- Service networking
- Traffic forwarding
- Load balancing to Pods

It programs networking rules so that traffic sent to a Service reaches one of the backend Pods.

---

# 15. CoreDNS

⭐⭐⭐⭐⭐

CoreDNS provides DNS inside Kubernetes.

Example

Application

```
frontend
```

calls

```
backend.default.svc.cluster.local
```

CoreDNS resolves the Service name to its ClusterIP.

---

# 16. DNS Flow

```
Pod

↓

CoreDNS

↓

Service IP

↓

Backend Pods
```

Applications communicate using names rather than hardcoded IP addresses.

---

# 17. Ingress

Ingress manages HTTP and HTTPS routing into the cluster.

Example

```
example.com

↓

Ingress

↓

frontend-service
```

---

# 18. AWS Load Balancer Controller

⭐⭐⭐⭐⭐

The AWS Load Balancer Controller watches Ingress resources.

Flow

```
Ingress Created

↓

AWS Load Balancer Controller

↓

Creates ALB

↓

Creates Target Groups

↓

Registers Pods
```

---

# 19. ALB vs NLB

| Feature | ALB | NLB |
|----------|-----|-----|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP) |
| Host Routing | Yes | No |
| Path Routing | Yes | No |
| Web Applications | Excellent | Limited |
| TCP Applications | No | Excellent |

---

# 20. Traffic Flow

```
User

↓

ALB

↓

Ingress

↓

Service

↓

kube-proxy

↓

Pod
```

---

# 21. Internal Communication

```
Frontend Pod

↓

backend-service

↓

CoreDNS

↓

ClusterIP

↓

Backend Pod
```

---

# 22. Production Scenario 1

Problem

Pod cannot reach another Pod.

Investigation

- Service exists?
- Endpoints available?
- CoreDNS healthy?
- NetworkPolicy blocking traffic?
- Pod labels correct?

---

# 23. Production Scenario 2

Problem

Ingress created.

ALB not created.

Investigation

```bash
kubectl get ingress
kubectl describe ingress
kubectl get pods -n kube-system
```

Check

- AWS Load Balancer Controller
- IRSA
- IAM permissions
- Controller logs

---

# 24. Production Scenario 3

Problem

Pods remain Pending.

Reason

```
Failed to assign IP
```

Possible Causes

- ENI limit reached
- Secondary IP exhaustion
- CNI issue

---

# 25. Production Scenario 4

Problem

DNS resolution fails.

Investigation

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system deployment/coredns
```

Verify

- CoreDNS Pods
- Service name
- Namespace
- Network connectivity

---

# 26. Production Scenario 5

Problem

Application inaccessible from the Internet.

Investigation

- Ingress
- ALB
- Target Group health
- Security Groups
- Route Tables
- Service type
- Pod readiness

---

# 27. Best Practices

- Use ClusterIP for internal services.
- Use Ingress with ALB for HTTP/HTTPS.
- Use NLB for TCP workloads.
- Monitor CoreDNS.
- Keep worker nodes in private subnets.
- Use AWS Load Balancer Controller instead of legacy integrations.
- Size nodes appropriately for Pod IP requirements.

---

# 28. Common Mistakes

❌ Accessing Pods directly by IP.

---

❌ Forgetting to install the AWS Load Balancer Controller.

---

❌ Using NodePort in production unnecessarily.

---

❌ Ignoring ENI/IP limits.

---

❌ Incorrect Service selectors.

---

# 29. Interview Questions

## Question 1

How does a Pod get an IP address in EKS?

### Perfect Answer

The AWS VPC CNI allocates a secondary private IP address from an Elastic Network Interface (ENI) attached to the worker node. That IP is assigned directly to the Pod, making it a native member of the VPC network.

---

## Question 2

What is the AWS VPC CNI?

### Perfect Answer

The AWS VPC CNI is the Kubernetes networking plugin used by Amazon EKS. It assigns VPC IP addresses to Pods, manages ENIs and secondary IPs, and enables direct Pod-to-VPC communication.

---

## Question 3

What is CoreDNS?

### Perfect Answer

CoreDNS is Kubernetes' internal DNS server. It resolves Service names to ClusterIP addresses, allowing Pods to communicate using DNS rather than hardcoded IP addresses.

---

## Question 4

What is kube-proxy?

### Perfect Answer

kube-proxy runs on every worker node and configures networking rules so that traffic sent to a Kubernetes Service is forwarded to one of its healthy backend Pods.

---

## Question 5

Difference between ClusterIP and LoadBalancer?

### Perfect Answer

ClusterIP is used for internal communication within the cluster. LoadBalancer provisions an external AWS Load Balancer, allowing applications to be accessed from outside the cluster.

---

## Question 6

Why would Pods remain Pending even though CPU and memory are available?

### Perfect Answer

One possible reason is IP exhaustion. If the worker node has no available secondary IP addresses or has reached ENI limits, the AWS VPC CNI cannot assign an IP to the new Pod.

---

## Question 7

How does traffic reach a Pod?

### Perfect Answer

For an Internet-facing application, traffic flows through the Application Load Balancer, then the Ingress resource, then the Kubernetes Service, where kube-proxy forwards the request to one of the backend Pods.

---

## Question 8

Why use a Service instead of Pod IPs?

### Perfect Answer

Pod IPs are ephemeral and change when Pods are recreated. Services provide a stable virtual IP and DNS name, ensuring reliable communication between applications.

---

# 30. Amazon Cross Questions

### Question

Your ALB exists but always returns 503. Where do you start?

### Perfect Answer

I check the Target Group health, verify the Service endpoints, confirm Pods are Ready, inspect the Ingress configuration, and review the AWS Load Balancer Controller logs.

---

### Question

The Pod cannot resolve `backend.default.svc.cluster.local`. What do you check?

### Perfect Answer

I verify that CoreDNS is running, ensure the Service exists, confirm the namespace is correct, inspect DNS configuration inside the Pod, and review CoreDNS logs.

---

### Question

Can two Pods on different nodes communicate directly?

### Perfect Answer

Yes.

The AWS VPC CNI provides VPC-native networking, allowing Pods on different worker nodes to communicate directly using their assigned VPC IP addresses.

---

### Question

What happens if the AWS Load Balancer Controller is removed?

### Perfect Answer

Existing ALBs continue operating, but Kubernetes Ingress resources will no longer be reconciled. New ALBs will not be created, and updates to existing Ingress resources will not be applied until the controller is restored.

---

# 31. Hands-on Labs (To Perform Later)

## Lab 1

Deploy two applications and communicate using ClusterIP.

---

## Lab 2

Inspect CoreDNS.

---

## Lab 3

Install the AWS Load Balancer Controller.

---

## Lab 4

Create an Ingress resource and verify ALB creation.

---

## Lab 5

Break the Service selector and troubleshoot.

---

## Lab 6

Inspect ENIs attached to worker nodes.

---

## Lab 7

Create multiple Pods and observe IP allocation.

---

# 32. One-Page Revision

```
Internet
    │
   ALB
    │
Ingress
    │
 Service
    │
kube-proxy
    │
   Pod
    │
Application

DNS

Pod
 │
CoreDNS
 │
Service
 │
ClusterIP
```

Remember

- AWS VPC CNI
- ENI
- Secondary IPs
- Prefix Delegation
- CoreDNS
- kube-proxy
- ClusterIP
- NodePort
- LoadBalancer
- Ingress
- AWS Load Balancer Controller

---

# Think Like a Production Engineer

Don't think:

> "The application is not reachable."

Think:

> "Where did the request stop?"

Follow this path:

```
Internet
    │
ALB Healthy?
    │
Ingress Correct?
    │
Service Exists?
    │
Endpoints Available?
    │
Pod Ready?
    │
Application Listening?
```

For internal communication:

```
Pod
 │
DNS?
 │
CoreDNS?
 │
Service?
 │
Endpoints?
 │
Backend Pod?
```

Always isolate the exact layer where traffic stops before making changes.

# End of Part 3
