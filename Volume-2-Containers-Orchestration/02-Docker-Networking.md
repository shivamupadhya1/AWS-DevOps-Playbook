# Docker Networking

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 02

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand Docker networking architecture
- Explain all Docker network drivers
- Understand bridge networking
- Explain host networking
- Explain overlay networking
- Understand Docker DNS
- Explain port mapping
- Troubleshoot networking issues
- Answer Docker networking interview questions

---

# 1. Why Docker Networking?

Suppose we have:

```
Container A

↓

Frontend
```

```
Container B

↓

Backend
```

```
Container C

↓

MySQL
```

Question

How will they communicate?

Docker Networking solves this problem.

---

# 2. Docker Networking Architecture

```
Container

↓

Network Namespace

↓

Virtual Ethernet (veth)

↓

Docker Network

↓

Docker Host NIC

↓

Internet
```

Each container gets its own isolated network namespace.

---

# 3. Docker Network Drivers

Docker provides multiple network drivers.

```
Bridge

Host

Overlay

None

Macvlan
```

Each has a different purpose.

---

# 4. Bridge Network (Default)

⭐⭐⭐⭐⭐

Default network driver.

When you install Docker:

```bash
docker network ls
```

Example output

```
bridge

host

none
```

The default bridge network allows containers on the same Docker host to communicate.

Architecture

```
Container A

↓

docker0

↓

Container B
```

The bridge network is implemented using a Linux bridge.

---

# 5. docker0 Bridge

Docker creates

```
docker0
```

during installation.

Check

```bash
ip addr show docker0
```

Example

```
172.17.0.1
```

Every container connected to the default bridge receives an IP address from this subnet.

Example

```
Container A

172.17.0.2

Container B

172.17.0.3
```

---

# 6. Virtual Ethernet Pair (veth)

⭐⭐⭐⭐⭐

Each container gets one end of a virtual Ethernet pair.

```
Container

eth0

↓

veth

↓

docker0

↓

Host
```

One end exists inside the container.

The other end exists on the host.

This is how packets enter and leave containers.

---

# 7. User-Defined Bridge Network

Recommended over the default bridge.

Create

```bash
docker network create app-network
```

Run

```bash
docker run -d --network app-network nginx
```

Advantages

- Automatic DNS
- Better isolation
- Easier service communication

---

# 8. Docker DNS

⭐⭐⭐⭐⭐

Suppose

```
Frontend

Backend
```

Instead of connecting using IP

```
172.18.0.4
```

Docker provides built-in DNS.

Example

```
Frontend

↓

backend

↓

Resolved Automatically
```

No hardcoded IPs.

This works on user-defined bridge networks.

---

# 9. Port Mapping

⭐⭐⭐⭐⭐

Example

```bash
docker run -p 8080:80 nginx
```

Meaning

```
Host

8080

↓

Container

80
```

Browser

```
localhost:8080

↓

Nginx

Port 80
```

Syntax

```
HostPort:ContainerPort
```

---

# 10. Host Network

Example

```bash
docker run --network host nginx
```

Architecture

```
Container

↓

Host Network Stack
```

No bridge.

No NAT.

No port mapping.

Advantages

- Maximum performance
- Lower latency

Disadvantages

- Less isolation
- Port conflicts

---

# 11. None Network

Example

```bash
docker run --network none nginx
```

Container has

```
Loopback Only
```

No Internet.

No external communication.

Useful for security testing.

---

# 12. Overlay Network

⭐⭐⭐⭐⭐

Business Problem

Suppose

```
Host A

↓

Container A
```

```
Host B

↓

Container B
```

How will they communicate?

Overlay Network.

Architecture

```
Container

↓

Overlay

↓

Host

↓

Overlay

↓

Container
```

Commonly used in:

- Docker Swarm
- Kubernetes (through CNI plugins)

---

# 13. Macvlan Network

Container gets its own MAC address.

Example

```
Switch

↓

Container

↓

Real IP
```

Container behaves like a physical machine on the network.

Use Cases

- Legacy applications
- Network appliances
- Monitoring tools

---

# 14. Container Communication

Same Network

```
Container A

↓

Container B
```

Works directly.

Different Networks

Communication fails unless explicitly connected.

---

# 15. DNS Resolution

Check

```bash
cat /etc/resolv.conf
```

Docker automatically injects DNS configuration into containers.

On user-defined networks, container names resolve automatically.

---

# 16. Inspect Network

```bash
docker network inspect bridge
```

Useful information

- Subnet
- Gateway
- Connected Containers
- Driver

---

# 17. Connect Running Container

```bash
docker network connect app-network nginx
```

Disconnect

```bash
docker network disconnect app-network nginx
```

---

# 18. Production Architecture

```
Browser

↓

Host Port

↓

Docker Bridge

↓

Frontend

↓

Docker DNS

↓

Backend

↓

Docker DNS

↓

Database
```

---

# 19. Troubleshooting Scenario 1

Problem

Container cannot access another container.

Check

```bash
docker network ls

docker network inspect

docker inspect <container>
```

Possible Causes

- Different networks
- Wrong hostname
- Container stopped

---

# 20. Troubleshooting Scenario 2

Problem

Application works inside the container.

Browser cannot access it.

Check

```bash
docker ps
```

Verify

```
PORTS
```

Ensure port mapping is configured.

---

# 21. Troubleshooting Scenario 3

Problem

Container cannot access the Internet.

Check

```bash
docker exec -it <container> ping 8.8.8.8

ip route

cat /etc/resolv.conf
```

Possible Causes

- DNS issue
- Firewall
- Broken bridge network
- Host routing issue

---

# 22. Troubleshooting Scenario 4

Problem

Container cannot resolve another container by name.

Possible Causes

- Using default bridge
- Containers on different networks
- Incorrect service name

Solution

Use a user-defined bridge network.

---

# 23. Best Practices

- Prefer user-defined bridge networks.
- Avoid hardcoded container IPs.
- Use container names for communication.
- Expose only required ports.
- Isolate unrelated applications.
- Monitor network traffic.

---

# 24. Common Mistakes

❌ Using IP addresses.

IPs change.

Use DNS names.

---

❌ Publishing unnecessary ports.

Expose only required services.

---

❌ Running everything on the host network.

This reduces isolation and increases the risk of port conflicts.

---

# 25. Interview Questions

## Question 1

What is the default Docker network?

### Perfect Answer

The default Docker network is the **bridge** network. Containers connected to it can communicate using IP addresses, while user-defined bridge networks also provide automatic DNS-based name resolution.

---

## Question 2

Difference between Bridge and Host Network?

### Perfect Answer

Bridge networking provides isolation through a virtual bridge and supports port mapping.

Host networking shares the host network stack directly, offering better performance but less isolation and no port mapping.

---

## Question 3

What is Overlay Network?

### Perfect Answer

Overlay networking enables containers running on different Docker hosts to communicate over a virtual network. It is commonly used in Docker Swarm and container orchestration platforms.

---

## Question 4

How does `-p 8080:80` work?

### Perfect Answer

It maps port **8080** on the host to port **80** inside the container, allowing clients to access the containerized application through the host's IP address.

---

## Question 5

Why should you use a user-defined bridge network?

### Perfect Answer

User-defined bridge networks provide automatic DNS resolution, improved isolation, and easier communication between containers using service names instead of IP addresses.

---

# 26. Amazon Cross Questions

### Question

Why can two containers communicate using names but not IP addresses in some environments?

### Perfect Answer

Container IP addresses are dynamic and may change. Docker's embedded DNS on user-defined networks resolves container names automatically, making communication more reliable.

---

### Question

Can two containers use the same internal port?

### Perfect Answer

Yes.

Each container has its own network namespace, so multiple containers can listen on port 80 internally. The host ports must be unique if they are published.

---

### Question

Can a container access the host?

### Perfect Answer

Yes.

The approach depends on the operating system and Docker configuration. For example, on Docker Desktop, `host.docker.internal` provides access to the host. On Linux, other networking methods may be used depending on the setup.

---

### Question

Why does Kubernetes not use Docker networking directly?

### Perfect Answer

Kubernetes relies on the **Container Network Interface (CNI)** specification. Different CNI plugins (such as Calico, Cilium, or Flannel) provide networking features across the cluster.

---

# 27. Hands-on Lab

## Lab 1

Create a user-defined network

```bash
docker network create app-network
```

---

## Lab 2

Run frontend

```bash
docker run -dit --name frontend \
--network app-network nginx
```

---

## Lab 3

Run backend

```bash
docker run -dit --name backend \
--network app-network nginx
```

---

## Lab 4

Verify DNS

```bash
docker exec -it frontend ping backend
```

---

## Lab 5

Inspect network

```bash
docker network inspect app-network
```

---

## Lab 6

Run with port mapping

```bash
docker run -d \
-p 8080:80 \
nginx
```

Visit

```
http://localhost:8080
```

---

# 28. One-Page Revision

```
Container

↓

veth Pair

↓

docker0 Bridge

↓

Host NIC

↓

Internet
```

Remember

- Bridge
- Host
- Overlay
- None
- Macvlan
- docker0
- veth
- Docker DNS
- Port Mapping
- User-defined Networks

---

# 29. Think Like a Production Engineer

Don't think:

> "Docker networking connects containers."

Think:

> "Docker networking isolates workloads while enabling controlled communication using virtual interfaces, bridges, DNS, and network namespaces."

Whenever someone says:

```
Application Unreachable

↓

Container Communication Failed

↓

Port Not Accessible
```

Follow this troubleshooting path:

```
docker ps

↓

docker network ls

↓

docker network inspect

↓

docker inspect

↓

DNS Resolution

↓

Port Mapping

↓

Application Logs
```

---

# Key Takeaways

Docker networking is built on Linux networking primitives such as network namespaces, virtual Ethernet pairs, and bridges. Understanding how bridge networks, user-defined networks, Docker DNS, and port mapping work is essential for debugging container communication problems and for transitioning to Kubernetes networking concepts later in this playbook.

# End of Chapter 02
