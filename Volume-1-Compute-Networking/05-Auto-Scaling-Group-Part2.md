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
