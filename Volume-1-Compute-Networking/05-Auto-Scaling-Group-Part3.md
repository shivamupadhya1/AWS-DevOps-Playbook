# Amazon Auto Scaling Group (ASG)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 05
>
> Part 3 – Production Troubleshooting, Terraform & Interview Mastery

---

# 36. Production Troubleshooting Playbook

---

## Scenario 1

### Problem

```
CPU = 95%

But ASG is NOT launching new EC2.
```

### Troubleshooting Flow

```
CPU High

↓

CloudWatch Metric Increasing?

↓

YES

↓

CloudWatch Alarm Triggered?

↓

NO

↓

Check Alarm Threshold
```

If Alarm Triggered

```
↓

Scaling Policy Attached?

↓

No

↓

Attach Scaling Policy
```

If Scaling Policy Exists

```
↓

ASG Activity History

↓

Scaling Suspended?

↓

IAM Permission?

↓

Maximum Capacity Reached?
```

### Commands

```bash
aws autoscaling describe-auto-scaling-groups

aws autoscaling describe-scaling-activities
```

---

## Scenario 2

### Problem

New EC2 launches

But

Never becomes healthy.

### Investigation

```
EC2 Running

↓

Target Group

↓

Unhealthy
```

Check

- Security Group
- Health Check Path
- Health Check Port
- Application Started?
- User Data Logs
- Nginx/Tomcat Running?
- Application Port Listening?

Linux

```bash
systemctl status nginx

systemctl status tomcat

journalctl -xe

curl localhost:8080/health

ss -tulpn
```

---

## Scenario 3

### Problem

ASG keeps launching and terminating instances continuously.

### Possible Causes

- Wrong health check path
- Startup takes longer than health check interval
- Application crash
- Wrong security group
- Incorrect target group port

### Resolution

Increase

```
Health Check Grace Period
```

Example

```
300 Seconds
```

to allow the application enough time to start.

---

## Scenario 4

### Problem

Traffic increased.

ASG launched new EC2.

Still

Website slow.

### Investigation

Possible bottlenecks

```
Database

↓

Redis

↓

External API

↓

Disk I/O

↓

Network

↓

Application Thread Pool
```

Scaling EC2 alone does not solve every performance problem.

---

## Scenario 5

### Problem

Auto Scaling launched EC2.

Application version is old.

### Investigation

Check

```
Launch Template Version

↓

AMI Version

↓

Golden Image Updated?
```

Very common production issue.

---

# 37. Root Cause Analysis Example

Production Incident

```
11:00 AM

Traffic Spike

↓

ASG Launches EC2

↓

EC2 Healthy

↓

ALB Healthy

↓

Users Getting 500 Errors
```

### RCA

Application depended on

```
Database Migration
```

User Data

never executed.

New instances

booted

but

application configuration

was incomplete.

### Lessons

Always validate

- User Data
- Startup Scripts
- Application Logs

before considering the deployment successful.

---

# 38. Terraform Example

```hcl
resource "aws_launch_template" "web" {
  name_prefix   = "prod-web"

  image_id      = "ami-xxxxxxxx"

  instance_type = "t3.medium"

  vpc_security_group_ids = [
    aws_security_group.web.id
  ]

  iam_instance_profile {
    name = aws_iam_instance_profile.ec2.name
  }

  user_data = base64encode(file("userdata.sh"))
}

resource "aws_autoscaling_group" "web" {

  desired_capacity = 3

  min_size = 2

  max_size = 6

  vpc_zone_identifier = [
    subnet-1,
    subnet-2
  ]

  launch_template {

    id = aws_launch_template.web.id

    version = "$Latest"
  }

  target_group_arns = [
    aws_lb_target_group.web.arn
  ]
}
```

---

# 39. AWS CLI

Create ASG

```bash
aws autoscaling create-auto-scaling-group
```

Describe ASG

```bash
aws autoscaling describe-auto-scaling-groups
```

Update Desired Capacity

```bash
aws autoscaling set-desired-capacity
```

Start Instance Refresh

```bash
aws autoscaling start-instance-refresh
```

Suspend Scaling

```bash
aws autoscaling suspend-processes
```

Resume Scaling

```bash
aws autoscaling resume-processes
```

---

# 40. Hands-on Lab

Objective

Learn Auto Scaling end-to-end.

---

### Step 1

Launch

```
ALB

+

ASG

+

2 EC2
```

---

### Step 2

Deploy

Nginx

or

Spring Boot

---

### Step 3

Create Target Tracking Policy

```
CPU

50%
```

---

### Step 4

Generate Load

```bash
sudo yum install stress -y

stress --cpu 2 --timeout 300
```

Observe

CloudWatch

↓

Alarm

↓

Scaling Activity

↓

New EC2

---

### Step 5

Stop

Stress

Observe

Scale In

---

### Step 6

Update AMI

Create

Launch Template Version 2

Run

Instance Refresh

Observe

Rolling Replacement

---

# 41. ASG vs ECS vs Kubernetes

| Feature | ASG | ECS | Kubernetes |
|----------|-----|-----|------------|
| Scales | EC2 | Containers | Pods |
| Unit | Instance | Task | Pod |
| Load Balancer | ALB | ALB | Ingress/ALB |
| Health Check | EC2/ELB | ECS | Readiness/Liveness |
| Deployment | EC2 | ECS Service | Deployment |
| AWS Managed | Yes | Yes | EKS Managed Control Plane |

---

# 42. Amazon Interview Questions

---

## Question 1

Your ASG Desired Capacity is 3.

You manually terminate one EC2.

What happens?

### Perfect Answer

ASG detects that only two healthy instances remain. It immediately launches a replacement instance using the configured Launch Template until the desired capacity of three is restored.

---

## Question 2

CPU reached 95%.

ASG did not scale.

Where will you investigate?

### Perfect Answer

My troubleshooting order would be:

1. Verify the CloudWatch metric.
2. Check whether the CloudWatch alarm entered the ALARM state.
3. Verify the scaling policy is attached.
4. Check ASG activity history.
5. Verify Min/Desired/Max capacity.
6. Confirm scaling processes are not suspended.
7. Check IAM permissions if scaling actions failed.

---

## Question 3

EC2 is healthy.

Application is down.

Will ASG replace it?

### Perfect Answer

Only if ELB health checks are enabled and fail.

If only EC2 health checks are configured, ASG sees the virtual machine as healthy and will not replace it.

---

## Question 4

You updated the Launch Template.

Will existing EC2 automatically update?

### Perfect Answer

No.

Only newly launched EC2 instances use the updated Launch Template.

Existing instances continue running until replaced manually or through Instance Refresh.

---

## Question 5

Can one ASG span multiple Availability Zones?

### Perfect Answer

Yes.

This is a recommended best practice because it improves fault tolerance. If one Availability Zone becomes unavailable, ASG can launch replacement instances in the remaining Availability Zones.

---

## Question 6

Can an Auto Scaling Group have multiple instance types?

### Perfect Answer

Yes.

Using Mixed Instance Policies, ASG can launch different EC2 instance types and combine On-Demand with Spot Instances to improve cost optimization and availability.

---

## Question 7

Why should you avoid Desired = Minimum = Maximum?

### Perfect Answer

Because the group loses elasticity.

For example:

```
Min = 2

Desired = 2

Max = 2
```

The application cannot scale out during traffic spikes.

---

## Question 8

Why is Launch Template preferred over Launch Configuration?

### Perfect Answer

Launch Templates support versioning, newer EC2 features, Spot Instances, T-series Unlimited, multiple network interfaces, and are the recommended modern replacement for Launch Configurations.

---

## Question 9

During an Instance Refresh, how does AWS avoid downtime?

### Perfect Answer

ASG replaces instances gradually. A new instance is launched, passes health checks, joins the target group, and only then is an old instance terminated. This rolling approach maintains application availability.

---

## Question 10

Can ASG replace an instance that fails ELB health checks but passes EC2 health checks?

### Perfect Answer

Yes, provided ELB health checks are enabled for the Auto Scaling Group. This ensures ASG evaluates application health rather than just VM health.

---

# 43. Interview Traps

### Trap 1

Interviewer

> Can ASG scale based on memory?

Correct Answer

Not directly.

CloudWatch does not collect memory metrics by default.

Install the CloudWatch Agent and create a custom metric.

---

### Trap 2

Interviewer

> Can ASG create an AMI?

Correct Answer

No.

ASG consumes Launch Templates that reference AMIs.

---

### Trap 3

Interviewer

> ASG launched a new EC2.

Why isn't it receiving traffic?

Correct Answer

Check:

- Target Group registration
- Health checks
- Security Groups
- NACLs
- Route Tables
- Application startup
- ALB listener rules

---

# 44. One-Page Revision Sheet

```
Traffic ↑

↓

CloudWatch Metric

↓

Alarm

↓

Scaling Policy

↓

Auto Scaling Group

↓

Launch Template

↓

EC2

↓

Health Check

↓

Target Group

↓

ALB

↓

Users
```

Remember

- Launch Template
- Min / Desired / Max
- Target Tracking
- Step Scaling
- Scheduled Scaling
- Predictive Scaling
- Warm Pools
- Lifecycle Hooks
- Instance Refresh
- Mixed Instances
- ELB Health Checks

---

# 45. Key Takeaways

Auto Scaling Group is much more than an automatic EC2 launcher. It is the core service responsible for maintaining application availability, elasticity, and cost optimization. A strong DevOps engineer understands not only how to configure ASG, but also how to troubleshoot scaling failures, design highly available architectures, and safely roll out infrastructure updates using Launch Templates and Instance Refresh.
