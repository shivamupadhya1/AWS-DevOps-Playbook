# Amazon Elastic Kubernetes Service (EKS)

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 08
>
> Part 1 – EKS Fundamentals & Architecture

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain Amazon EKS architecture
- Understand the Kubernetes control plane
- Explain managed control plane
- Understand worker nodes
- Explain how Pods are scheduled
- Understand EKS networking
- Compare EKS with ECS
- Answer architecture interview questions confidently

---

# 1. What is Amazon EKS?

Amazon Elastic Kubernetes Service (EKS) is AWS's managed Kubernetes service.

AWS manages the Kubernetes control plane while customers manage the worker nodes (or use managed compute options such as managed node groups or Fargate for EKS).

Instead of installing Kubernetes manually, AWS provides a production-ready Kubernetes cluster.

---

# 2. Why EKS?

Without EKS

```
Install Kubernetes

↓

Install etcd

↓

Install API Server

↓

Install Scheduler

↓

Install Controller Manager

↓

High Availability

↓

Upgrades

↓

Backups

↓

Certificates
```

Everything is your responsibility.

---

With EKS

AWS manages

```
API Server

etcd

Scheduler

Controller Manager

High Availability

Upgrades (control plane)

Patching (control plane)
```

You focus on workloads.

---

# 3. EKS Architecture

```
Developer

↓

kubectl

↓

API Server

↓

Scheduler

↓

Worker Nodes

↓

Pods
```

AWS manages

```
API Server

Scheduler

Controller Manager

etcd
```

You manage

```
Applications

Pods

Node Groups

Networking

IAM

Storage
```

---

# 4. High-Level Architecture

```
                  AWS Managed

          Kubernetes Control Plane

 ┌──────────────────────────────────────┐
 │ API Server                           │
 │ Scheduler                            │
 │ Controller Manager                   │
 │ etcd                                │
 └──────────────────────────────────────┘
                 │
─────────────────┼──────────────────────────
                 │
          Worker Nodes

        Node 1      Node 2

          │            │

       Pods         Pods
```

---

# 5. What Does AWS Manage?

AWS manages:

- Kubernetes API Server
- etcd
- Scheduler
- Controller Manager
- Control plane upgrades
- High availability
- Control plane backups
- Certificates

---

# 6. What Do You Manage?

You manage:

- Worker Nodes
- Pods
- Deployments
- Services
- Ingress
- IAM
- Security Groups
- EBS Volumes
- Applications

---

# 7. Control Plane

⭐⭐⭐⭐⭐

The Control Plane is the brain of Kubernetes.

It decides:

- Where Pods run
- Desired state
- Scaling
- Scheduling
- Health monitoring

Without it, the cluster cannot function.

---

# 8. API Server

⭐⭐⭐⭐⭐

The API Server is the entry point into Kubernetes.

Everything communicates with it.

Example

```
kubectl apply

↓

API Server

↓

Cluster
```

Even internal Kubernetes components use the API Server.

---

# 9. etcd

⭐⭐⭐⭐⭐

etcd is Kubernetes' distributed key-value database.

It stores:

- Deployments
- Pods
- Services
- ConfigMaps
- Secrets
- Node information
- Cluster configuration

Think of etcd as the cluster database.

---

# 10. Scheduler

⭐⭐⭐⭐⭐

The Scheduler decides:

```
Which Pod

↓

Should Run

↓

On Which Node
```

It evaluates:

- Available CPU
- Available memory
- Node selectors
- Affinity/Anti-affinity
- Taints & Tolerations
- Resource requests

---

# 11. Controller Manager

The Controller Manager continuously compares:

Desired State

vs

Current State

Example

Desired

```
5 Pods
```

Current

```
4 Pods
```

Controller creates one more Pod.

---

# 12. Worker Nodes

Worker Nodes are EC2 instances (or managed compute) where Pods run.

Each worker contains:

```
kubelet

Container Runtime

kube-proxy

Pods
```

---

# 13. kubelet

⭐⭐⭐⭐⭐

kubelet is the Kubernetes agent running on every worker node.

Responsibilities

- Register node
- Receive Pod instructions
- Start containers
- Monitor Pods
- Report status

---

# 14. Container Runtime

Responsible for running containers.

Examples

- containerd
- Other OCI-compatible runtimes

It starts and stops containers as instructed by kubelet.

---

# 15. kube-proxy

Responsible for Kubernetes networking.

Functions

- Service networking
- Traffic forwarding
- Load balancing between Pods

---

# 16. Pod Scheduling Flow

```
Deployment

↓

ReplicaSet

↓

Pod

↓

API Server

↓

Scheduler

↓

Node Selected

↓

kubelet

↓

Container Runtime

↓

Running Pod
```

---

# 17. Complete Request Flow

```
Developer

↓

kubectl apply

↓

API Server

↓

etcd

↓

Scheduler

↓

Worker Node

↓

kubelet

↓

Container Runtime

↓

Pod Running
```

---

# 18. Why Companies Prefer EKS

Benefits

- Managed Kubernetes
- Highly available control plane
- Easy upgrades
- AWS integrations
- IAM integration
- Security
- Monitoring
- Reduced operational effort

---

# 19. EKS vs ECS

| Feature | ECS | EKS |
|----------|-----|-----|
| Orchestrator | AWS ECS | Kubernetes |
| Portability | AWS-focused | Multi-cloud |
| Learning Curve | Easier | Steeper |
| Kubernetes APIs | No | Yes |
| CNCF Ecosystem | No | Yes |
| Best For | AWS-native apps | Kubernetes platforms |

---

# 20. Production Example

Company has

```
AWS

Azure

GCP
```

Which service?

EKS.

Reason

Kubernetes knowledge transfers across cloud providers, making migrations and multi-cloud strategies easier.

---

# 21. Best Practices

- Use Managed Node Groups unless you have a specific requirement.
- Use IAM Roles for Service Accounts (IRSA).
- Keep worker nodes in private subnets.
- Use ALB Ingress Controller.
- Monitor with CloudWatch and Prometheus.
- Keep Kubernetes versions supported.

---

# 22. Common Mistakes

❌ Exposing worker nodes publicly.

---

❌ Giving worker node IAM role excessive permissions.

---

❌ Running everything in the default namespace.

---

❌ Ignoring resource requests and limits.

---

# 23. Interview Questions

## Question 1

What is Amazon EKS?

### Perfect Answer

Amazon EKS is AWS's managed Kubernetes service. AWS manages the Kubernetes control plane while customers manage worker nodes and Kubernetes workloads.

---

## Question 2

What components does AWS manage in EKS?

### Perfect Answer

AWS manages the Kubernetes control plane, including the API Server, etcd, Scheduler, Controller Manager, high availability, certificates, and control plane upgrades.

---

## Question 3

What components do you manage?

### Perfect Answer

I manage worker nodes, Pods, Deployments, Services, Ingress, storage, networking, IAM configuration, and the applications running on the cluster.

---

## Question 4

What is the API Server?

### Perfect Answer

The API Server is the central entry point to Kubernetes. All cluster operations, whether initiated by users or internal components, are processed through it.

---

## Question 5

What is etcd?

### Perfect Answer

etcd is Kubernetes' distributed key-value datastore that stores the entire cluster state, including Pods, Deployments, Services, ConfigMaps, Secrets, and other Kubernetes objects.

---

## Question 6

What does the Scheduler do?

### Perfect Answer

The Scheduler selects the most appropriate worker node for a Pod based on resource availability, scheduling policies, affinity rules, taints, tolerations, and other constraints.

---

## Question 7

What does kubelet do?

### Perfect Answer

kubelet runs on every worker node. It communicates with the API Server, ensures Pods are running as expected, starts containers through the container runtime, and reports node and Pod health.

---

## Question 8

Difference between ECS and EKS?

### Perfect Answer

ECS is AWS's native container orchestration service, while EKS provides managed Kubernetes. EKS is preferred when Kubernetes portability, ecosystem compatibility, or multi-cloud strategies are important.

---

# 24. Amazon Cross Questions

### Question

Why would a company migrate from ECS to EKS?

### Perfect Answer

A company may migrate to EKS to standardize on Kubernetes, improve portability across cloud providers, leverage the Kubernetes ecosystem, or align with existing Kubernetes expertise.

---

### Question

If the Scheduler is down, can new Pods be scheduled?

### Perfect Answer

No.

Existing Pods continue running, but new Pods cannot be scheduled until the Scheduler becomes available.

---

### Question

If etcd becomes unavailable, what happens?

### Perfect Answer

The control plane cannot reliably read or update cluster state. Existing workloads may continue running temporarily, but cluster management operations are affected.

---

# 25. Hands-on Labs (To Perform Later)

## Lab 1

Create an EKS cluster using eksctl.

---

## Lab 2

Connect using kubectl.

---

## Lab 3

Deploy an NGINX Deployment.

---

## Lab 4

Expose it with a Service.

---

## Lab 5

Inspect Pods and Nodes.

---

# 26. One-Page Revision

```
kubectl
   │
API Server
   │
etcd
   │
Scheduler
   │
Worker Node
   │
kubelet
   │
Container Runtime
   │
Pod
```

Remember

- Control Plane
- API Server
- etcd
- Scheduler
- Controller Manager
- Worker Nodes
- kubelet
- kube-proxy
- Container Runtime

---

# 27. Think Like a Production Engineer

Don't think:

> "A Pod just runs."

Think:

> "A Pod reaches the Running state only after multiple control-plane components coordinate successfully."

Mental Flow

```
kubectl

↓

API Server

↓

etcd

↓

Scheduler

↓

Node

↓

kubelet

↓

Container Runtime

↓

Pod Running
```

# End of Part 1
