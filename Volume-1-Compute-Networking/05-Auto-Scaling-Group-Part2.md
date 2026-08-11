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
