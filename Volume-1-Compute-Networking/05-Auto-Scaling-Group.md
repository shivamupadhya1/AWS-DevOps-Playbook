# Amazon Auto Scaling Group (ASG)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 05
>
> Part 1

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain why AWS introduced Auto Scaling.
- Understand Launch Templates.
- Explain Desired, Minimum, and Maximum Capacity.
- Understand the complete request flow.
- Design highly available architectures.
- Answer senior DevOps interview questions.

---

# 1. Business Problem

Imagine your company has an e-commerce website.

Normal Day

```
500 Users
```

One EC2

```
4 vCPU

16 GB RAM
```

Everything works.

---

Now

Black Friday Sale

```
500

↓

50,000 Users
```

Question

Can one EC2 handle this traffic?

No.

CPU

```
100%
```

Memory

```
95%
```

Application

```
Slow

↓

Crash
```

Now imagine

After sale

Traffic again becomes

```
500 Users
```

Question

Should we keep paying for

20 EC2 instances?

No.

AWS needed a service which could

- Automatically add servers
- Automatically remove servers
- Maintain availability
- Reduce infrastructure cost

Hence,

AWS introduced

# Auto Scaling Group (ASG)

---

# 2. What is Auto Scaling Group?

Auto Scaling Group is a service that automatically

- Launches EC2 instances
- Terminates EC2 instances
- Maintains desired capacity
- Replaces unhealthy instances

based on demand.

Think of ASG as a manager.

It doesn't run applications.

It manages EC2 instances.

---

# 3. Internal Working

```
Users

↓

ALB

↓

Target Group

↓

Auto Scaling Group

↓

Launch Template

↓

EC2
```

Notice

Auto Scaling

does not create EC2 from scratch.

It always launches EC2 using a

Launch Template.

---

# 4. Components of ASG

---

## Launch Template

This is one of the most asked interview topics.

Launch Template contains

```
AMI

↓

Instance Type

↓

Security Group

↓

IAM Role

↓

User Data

↓

EBS

↓

Network Configuration
```

Question

Can ASG launch an EC2 without Launch Template?

No.

---

## Auto Scaling Group

Contains

```
Minimum

Desired

Maximum
```

capacity.

It manages

all EC2 instances.

---

## CloudWatch

Provides metrics.

Example

```
CPU

Memory

Network

Custom Metrics
```

These metrics trigger scaling policies.

---

## Target Group

Keeps track of

Healthy

and

Unhealthy

instances.

---

## Application Load Balancer

Distributes traffic

between EC2 instances.

---

# 5. Request Flow

```
User

↓

Route53

↓

ALB

↓

Target Group

↓

ASG

↓

EC2

↓

Application
```

If

CPU increases

↓

CloudWatch Alarm

↓

Scaling Policy

↓

ASG

↓

Launch Template

↓

New EC2

↓

Register with Target Group

↓

ALB starts routing traffic
```

---

# 6. Desired Capacity

Suppose

```
Desired = 3
```

ASG always tries to keep

```
3 Running EC2
```

If

one EC2 crashes

```
3

↓

2
```

ASG automatically launches

one more EC2.

Desired again becomes

```
3
```

---

# 7. Minimum Capacity

Example

```
Minimum = 2
```

Even if

traffic becomes zero

ASG never goes below

```
2 EC2
```

---

# 8. Maximum Capacity

Example

```
Maximum = 10
```

If CPU becomes

```
100%
```

ASG can launch

new EC2

until

```
10 Instances
```

After that

Scaling stops.

---

# 9. Capacity Example

```
Minimum = 2

Desired = 4

Maximum = 10
```

Current

```
4 Running EC2
```

Traffic increases.

CloudWatch Alarm triggers.

ASG launches

```
5

↓

6

↓

7

↓

8
```

Until

load decreases

or

Maximum

is reached.

---

# 10. Health Checks

ASG supports

---

## EC2 Health Check

Checks

```
Instance Status
```

If EC2 fails

Replace.

---

## ELB Health Check

Checks

```
Application Health
```

Example

```
/health
```

If endpoint fails

Instance replaced.

Production always uses

ELB Health Checks.

---

# 11. Real Production Story

A payment application

always maintained

```
Desired = 6
```

One EC2 crashed

at midnight.

Nobody noticed.

Why?

Because

ASG detected

```
6

↓

5
```

Immediately

```
Launch Template

↓

New EC2

↓

Healthy

↓

Desired = 6
```

Application remained available.

---

# 12. Best Practices

✅ Use Launch Templates.

✅ Never manually launch production EC2.

✅ Use ALB.

✅ Enable ELB Health Checks.

✅ Keep Desired Capacity greater than one.

✅ Use IAM Roles.

✅ Use CloudWatch monitoring.

---

# 13. Common Mistakes

❌ Desired = 1

One EC2 failure

↓

Application Down.

---

❌ Maximum = Desired

System

cannot scale.

---

❌ Using public AMIs.

Use

Golden AMIs.

---

❌ Not using Health Checks.

---

# 14. Interview Questions

---

## Question 1

Why did AWS introduce Auto Scaling?

### Perfect Answer

AWS introduced Auto Scaling to automatically adjust the number of EC2 instances based on application demand. This improves availability during traffic spikes, reduces infrastructure cost during low traffic, and eliminates manual intervention for scaling operations.

---

## Question 2

Difference between Desired, Minimum and Maximum Capacity?

### Perfect Answer

Minimum Capacity defines the lowest number of instances ASG will maintain.

Desired Capacity is the target number of running instances.

Maximum Capacity limits how many instances ASG can launch during scale-out.

Example:

```
Minimum = 2

Desired = 4

Maximum = 10
```

Normally

4 EC2 run.

ASG can never go below 2

and never above 10.

---

## Question 3

What happens if one EC2 crashes?

### Perfect Answer

ASG continuously monitors instance health. If an instance becomes unhealthy based on EC2 or ELB health checks, ASG terminates it and launches a replacement instance using the Launch Template to maintain the desired capacity.

---

## Question 4

Can ASG launch EC2 without AMI?

### Perfect Answer

No.

Every EC2 launched by ASG is created from the Launch Template, which references an AMI. Without an AMI, ASG cannot provision a new instance.

---

## Question 5

Can we manually terminate an EC2 inside an Auto Scaling Group?

### Perfect Answer

Yes.

However,

ASG detects

```
Desired = 3

Running = 2
```

and automatically launches a new EC2 to restore the desired capacity.

---

# 15. Amazon Cross Questions

---

### Question

Why does ASG use Launch Templates instead of configuring servers after launch?

### Perfect Answer

Using Launch Templates ensures every EC2 instance is created with identical configuration, including the operating system, instance type, IAM role, security groups, user data, and storage settings. This provides consistency, faster provisioning, and predictable deployments.

---

### Question

Can ASG work without an ALB?

### Perfect Answer

Yes.

ASG can function independently and maintain the desired number of EC2 instances. However, for web applications, an ALB is typically used to distribute traffic and perform application-level health checks.

---

# 16. Think Like a Production Engineer

Don't think

> ASG launches EC2.

Think

> ASG maintains application availability.

Its real responsibility is

```
Desired Capacity

↓

Healthy Instances

↓

Business Continuity
```

---

# Revision Sheet

Remember

- ASG manages EC2.
- Uses Launch Templates.
- Maintains Desired Capacity.
- Uses CloudWatch metrics.
- Supports EC2 and ELB health checks.
- Replaces unhealthy instances automatically.

```
Traffic ↑

↓

CloudWatch

↓

Scaling Policy

↓

ASG

↓

Launch Template

↓

New EC2

↓

Target Group

↓

ALB
```

---

# End of Part 1
# Amazon Auto Scaling Group (ASG)

> AWS DevOps Playbook
>
> Volume 1 – Compute & Networking
>
> Chapter 05
>
> Part 2 – Scaling Policies, Warm Pools & Production

---

# 17. Scaling Policies

Scaling Policy tells the Auto Scaling Group **when** to launch new EC2 instances and **when** to terminate them.

Think of ASG as a worker.

Scaling Policy is the manager giving instructions.

```
CloudWatch

↓

Alarm

↓

Scaling Policy

↓

ASG

↓

Launch/Terminate EC2
```

---

# 18. Target Tracking Scaling

⭐⭐⭐⭐⭐ (Most Asked)

Suppose

```
Target CPU = 60%
```

Current

```
2 EC2

CPU = 95%
```

CloudWatch continuously monitors CPU.

```
95%

↓

Above Target

↓

Launch New EC2

↓

Traffic Distributed

↓

CPU Drops
```

Later

```
CPU = 20%
```

ASG automatically removes unnecessary EC2 instances.

---

### Perfect Interview Answer

Target Tracking automatically adjusts the number of EC2 instances to maintain a target metric such as CPU utilization or ALB request count. AWS continuously monitors the metric and scales out or scales in without manually defining thresholds.

---

# 19. Step Scaling

Used when scaling should happen in **steps**.

Example

```
CPU > 60%

↓

Add 1 EC2

CPU > 75%

↓

Add 2 EC2

CPU > 90%

↓

Add 4 EC2
```

This is useful for workloads where demand increases rapidly.

---

### Production Example

Gaming Application

```
70%

↓

Add 2

90%

↓

Add 6

95%

↓

Add 10
```

---

# 20. Simple Scaling

Old scaling method.

Example

```
CPU > 70%

↓

Launch 1 EC2

↓

Wait 5 Minutes

↓

Check Again
```

Because it waits between actions, it reacts more slowly.

AWS generally recommends **Target Tracking** or **Step Scaling** for new workloads.

---

# 21. Scheduled Scaling

Sometimes traffic is predictable.

Example

```
Office Application

Morning

9 AM

↓

Traffic High

Night

10 PM

↓

Traffic Low
```

Schedule

```
8:45 AM

Desired = 10

10:15 PM

Desired = 2
```

No CloudWatch alarm is required because scaling is based on time.

---

# 22. Predictive Scaling

AWS analyzes historical CloudWatch metrics.

Example

```
Every Monday

10 AM

Traffic Spike
```

AWS predicts the spike and starts launching EC2 instances **before** traffic arrives.

Best suited for highly predictable workloads.

---

# 23. Warm Pools

⭐⭐⭐⭐⭐

One of the most overlooked interview topics.

Business Problem

Launching a new EC2 can take:

```
2–5 Minutes
```

During a sudden traffic spike, those minutes matter.

AWS introduced **Warm Pools**.

```
ASG

↓

Stopped EC2

↓

Warm Pool

↓

Traffic Spike

↓

Start Warm Instance

↓

Serve Traffic Faster
```

Instead of creating a new server from scratch, ASG starts a pre-initialized instance.

---

### Production Example

```
Black Friday Sale

↓

Traffic x10

↓

Warm Pool

↓

Instances Available Quickly
```

---

# 24. Lifecycle Hooks

Normally

```
Launch EC2

↓

Healthy

↓

Traffic Starts
```

But sometimes we need extra steps.

Example

- Install monitoring agent
- Download configuration
- Register with CMDB
- Run validation scripts

Lifecycle Hooks pause the instance before it starts serving traffic.

```
Launch EC2

↓

Lifecycle Hook

↓

Configuration

↓

Health Check

↓

Register to Target Group

↓

Traffic
```

---

# 25. Instance Refresh

Suppose

```
AMI v1

↓

Java 17
```

Now

```
AMI v2

↓

Java 21
```

Should you terminate every EC2 manually?

No.

Use **Instance Refresh**.

```
Old Instance

↓

Terminate One

↓

Launch New

↓

Healthy

↓

Next Instance
```

This provides a rolling replacement with minimal disruption.

---

# 26. Mixed Instance Policy

Large production environments often use both:

```
On-Demand

+

Spot Instances
```

Example

```
70%

On-Demand

30%

Spot
```

Benefits

- Lower cost
- Better availability
- Reduced Spot interruption impact

---

# 27. Health Check Deep Dive

### EC2 Health Check

Checks

- Hardware
- Hypervisor
- Operating System status

### ELB Health Check

Checks

```
HTTP GET

/health
```

Returns

```
200 OK
```

Only then is the instance considered healthy for traffic.

---

### Which Should You Use?

For production web applications,

Always enable

```
EC2 Health Check

+

ELB Health Check
```

EC2 confirms the machine is running.

ELB confirms the application is actually responding.

---

# 28. Production Scenario 1

Problem

```
CPU

95%
```

Traffic

Increasing.

Question

What happens?

### Perfect Answer

CloudWatch alarm triggers.

↓

Scaling Policy executes.

↓

ASG launches new EC2.

↓

Instance boots using Launch Template.

↓

Passes health checks.

↓

Registers with Target Group.

↓

ALB starts routing traffic.

---

# 29. Production Scenario 2

Problem

```
EC2

Healthy

Application

Down
```

Question

Will ASG replace it?

### Perfect Answer

If only EC2 health checks are enabled, ASG may not replace the instance because the VM itself is healthy.

If ELB health checks fail, ASG marks the instance unhealthy and launches a replacement.

This is why production systems should use ELB health checks.

---

# 30. Production Scenario 3

Problem

Traffic increases rapidly.

New EC2 instances take

```
5 Minutes
```

Users experience failures.

### Solution

Use

- Warm Pools
- Predictive Scaling
- Better AMIs (pre-installed software)

---

# 31. Production Scenario 4

Security team releases a critical patch.

Question

How do you update 100 EC2 instances?

### Perfect Answer

1. Build a new Golden AMI.
2. Update the Launch Template.
3. Start Instance Refresh.
4. Replace instances gradually.
5. Verify health after each replacement.

Never patch all production servers manually.

---

# 32. Best Practices

✅ Use Launch Templates instead of Launch Configurations.

✅ Prefer Target Tracking for most workloads.

✅ Use ELB health checks.

✅ Maintain Golden AMIs.

✅ Enable detailed CloudWatch monitoring.

✅ Configure Warm Pools for latency-sensitive applications.

✅ Use Instance Refresh for upgrades.

---

# 33. Common Mistakes

❌ Scaling based only on CPU.

Sometimes the bottleneck is:

- Database
- Disk I/O
- Network
- External API

---

❌ Forgetting cooldown or stabilization behavior.

Rapid scale-in/scale-out can cause instability.

---

❌ Using old AMIs.

Every new instance launches with outdated software.

---

# 34. Interview Questions

---

## Question 1

Difference between Target Tracking and Step Scaling?

### Perfect Answer

Target Tracking automatically maintains a target metric (such as 60% CPU) by continuously adjusting capacity.

Step Scaling increases or decreases capacity in predefined steps based on metric ranges.

Target Tracking is easier to manage for most applications.

---

## Question 2

When would you use Scheduled Scaling?

### Perfect Answer

Scheduled Scaling is used when traffic patterns are predictable, such as office hours, monthly payroll processing, or planned marketing campaigns.

---

## Question 3

What problem do Warm Pools solve?

### Perfect Answer

Warm Pools reduce scale-out time by keeping pre-initialized EC2 instances ready to start. This minimizes delays during sudden traffic spikes.

---

## Question 4

What is Instance Refresh?

### Perfect Answer

Instance Refresh gradually replaces EC2 instances in an Auto Scaling Group with new instances created from an updated Launch Template or AMI, reducing deployment risk and avoiding full downtime.

---

## Question 5

Why are ELB health checks better than EC2 health checks?

### Perfect Answer

EC2 health checks verify the virtual machine is operational.

ELB health checks verify that the application is actually responding to requests.

A healthy VM does not necessarily mean a healthy application.

---

# 35. Think Like a Production Engineer

Don't think

> "ASG adds servers."

Think

> "ASG maintains business availability while balancing cost."

Whenever someone says

```
Traffic Increased
```

Your thought process should be

```
CloudWatch

↓

Scaling Policy

↓

Launch Template

↓

ASG

↓

Health Check

↓

Target Group

↓

ALB

↓

Users
```

---

# End of Part 2
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
