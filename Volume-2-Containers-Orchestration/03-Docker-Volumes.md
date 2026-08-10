# Docker Volumes

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 03

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand why Docker volumes are needed.
- Differentiate between Named Volumes, Bind Mounts, and tmpfs.
- Explain Docker Volume Drivers.
- Understand persistent storage.
- Backup and restore Docker volumes.
- Troubleshoot storage issues.
- Answer Docker storage interview questions confidently.

---

# 1. Why Do We Need Docker Volumes?

By default, Docker containers are **ephemeral**.

This means:

```
Container Created

↓

Application Writes Data

↓

Container Deleted

↓

Data Lost
```

Example:

You run MySQL inside Docker.

```
docker run mysql
```

Create a database.

Delete the container.

Create another MySQL container.

The database is gone.

Why?

Because the data was stored inside the container's writable layer.

---

# 2. Docker Writable Layer

Recall from Chapter 1:

```
Image (Read Only)

↓

Writable Layer

(Container)
```

Every write operation goes into the writable layer.

Examples:

```
/var/log

/tmp

/home

/etc
```

Problem:

If the container disappears, the writable layer disappears too.

---

# 3. Solution – Docker Volumes

Docker Volumes store data **outside** the container.

```
Application

↓

Container

↓

Volume

↓

Host Storage
```

Delete the container.

Volume still exists.

Create a new container.

Attach the same volume.

Data returns.

---

# 4. Types of Docker Storage

Docker supports three primary storage mechanisms.

```
Named Volume

Bind Mount

tmpfs
```

Each has different use cases.

---

# 5. Named Volumes

⭐⭐⭐⭐⭐

Named Volumes are managed by Docker.

Create

```bash
docker volume create mysql-data
```

Run

```bash
docker run -d \
--name mysql \
-v mysql-data:/var/lib/mysql \
mysql:8
```

Architecture

```
MySQL Container

↓

/var/lib/mysql

↓

Named Volume

↓

Docker Managed Storage
```

Advantages

- Easy to manage
- Portable
- Safer than bind mounts
- Preferred for production containers

---

# 6. Bind Mounts

Instead of Docker managing storage,

you choose a directory.

Example

```bash
docker run -d \
-v /home/ubuntu/app:/app \
nginx
```

Architecture

```
Host Directory

↓

/home/ubuntu/app

↓

Container

↓

/app
```

Changes on the host appear immediately inside the container.

Advantages

- Excellent for development
- Easy code editing
- Direct host access

Disadvantages

- Host dependent
- Permissions issues
- Less portable

---

# 7. tmpfs Mount

tmpfs stores data **only in memory**.

Example

```bash
docker run \
--tmpfs /tmp \
nginx
```

Architecture

```
RAM

↓

tmpfs

↓

Container
```

When the container stops,

the data disappears.

Use Cases

- Temporary files
- Secrets
- Session data
- High-speed caching

---

# 8. Volume Drivers

⭐⭐⭐⭐⭐

Docker supports external storage.

Examples

```
Local

NFS

Amazon EFS

Azure Files

NetApp

CSI Plugins
```

Architecture

```
Container

↓

Volume Driver

↓

Storage System
```

This enables persistent storage across multiple hosts.

---

# 9. Inspect Volumes

List

```bash
docker volume ls
```

Inspect

```bash
docker volume inspect mysql-data
```

Example Output

```
Driver

local

Mountpoint

/var/lib/docker/volumes/...
```

---

# 10. Remove Volumes

Delete

```bash
docker volume rm mysql-data
```

Delete unused volumes

```bash
docker volume prune
```

Be careful.

Deleted data cannot be recovered unless backed up.

---

# 11. Anonymous Volumes

Example

```bash
docker run \
-v /var/lib/mysql \
mysql
```

Docker automatically creates an unnamed volume.

Example

```
c8d0f3b...
```

These are harder to manage.

Production Recommendation

Prefer **Named Volumes**.

---

# 12. Sharing Volumes

Multiple containers can share the same volume.

Example

```
Container A

↓

Shared Volume

↑

Container B
```

Useful for

- Shared logs
- Shared configuration
- Shared static files

---

# 13. Read-Only Mounts

Example

```bash
docker run \
-v config-data:/app/config:ro \
nginx
```

The container can read

but cannot modify files.

Improves security.

---

# 14. Backup a Volume

One approach:

```bash
docker run --rm \
-v mysql-data:/data \
-v $(pwd):/backup \
ubuntu \
tar czf /backup/mysql-backup.tar.gz /data
```

Result

```
mysql-backup.tar.gz
```

---

# 15. Restore a Volume

Example

```bash
docker run --rm \
-v mysql-data:/data \
-v $(pwd):/backup \
ubuntu \
tar xzf /backup/mysql-backup.tar.gz -C /
```

---

# 16. Production Architecture

```
Application

↓

Container

↓

Named Volume

↓

Amazon EBS

↓

Disk
```

OR

```
Application

↓

Container

↓

EFS

↓

Multiple Containers
```

---

# 17. Database Example

MySQL

```
Container

↓

/var/lib/mysql

↓

Named Volume
```

Delete Container

↓

Create New Container

↓

Attach Same Volume

↓

Database Still Exists

---

# 18. Development Example

```
VS Code

↓

Host Directory

↓

Bind Mount

↓

Container
```

Edit code on the host.

Immediately visible inside the container.

---

# 19. Production Scenario 1

Problem

Database disappeared after recreating the container.

Root Cause

No persistent volume was attached.

Solution

Use a named volume or external storage.

---

# 20. Production Scenario 2

Problem

Application cannot write files.

Possible Causes

- Host permissions
- Read-only mount
- Wrong ownership
- SELinux/AppArmor restrictions

Check

```bash
ls -l

docker inspect

docker volume inspect
```

---

# 21. Production Scenario 3

Problem

Two containers cannot see the same files.

Check

- Are both using the same volume?
- Are they mounting the same path?
- Are permissions correct?

---

# 22. Production Scenario 4

Problem

Container deleted accidentally.

Question

Can data be recovered?

Answer

If stored in a Docker volume,

Yes.

If stored only in the writable layer,

No.

---

# 23. Best Practices

- Prefer Named Volumes for persistent application data.
- Use Bind Mounts mainly for development.
- Use tmpfs only for temporary in-memory data.
- Backup important volumes regularly.
- Use read-only mounts where appropriate.
- Avoid storing databases in the writable layer.

---

# 24. Common Mistakes

❌ Running databases without volumes.

Risk:

Permanent data loss.

---

❌ Using bind mounts for production databases.

Prefer Docker-managed volumes or network storage.

---

❌ Deleting volumes without backups.

Always verify before running

```bash
docker volume prune
```

---

❌ Storing secrets in bind-mounted directories.

Use a dedicated secrets management solution.

---

# 25. Interview Questions

## Question 1

Why do we need Docker Volumes?

### Perfect Answer

Docker volumes provide persistent storage outside the container's writable layer. They allow data to survive container deletion and recreation.

---

## Question 2

Difference between Named Volume and Bind Mount?

### Perfect Answer

Named Volumes are managed by Docker and are portable across environments.

Bind Mounts map a specific host directory into the container and are mainly used for development or when direct host access is required.

---

## Question 3

What is tmpfs?

### Perfect Answer

tmpfs is an in-memory filesystem. Data is stored in RAM and is lost when the container stops. It is useful for temporary or sensitive data.

---

## Question 4

Can two containers share the same volume?

### Perfect Answer

Yes.

Multiple containers can mount the same Docker volume, allowing them to share files and data, provided the application supports concurrent access.

---

## Question 5

Where are Docker volumes stored?

### Perfect Answer

By default, Docker stores local volumes under:

```text
/var/lib/docker/volumes/
```

The exact location can vary depending on the storage driver and Docker configuration.

---

# 26. Amazon Cross Questions

### Question

If a container is deleted, is the Docker volume also deleted?

### Perfect Answer

No.

Volumes exist independently of containers and remain until they are explicitly removed.

---

### Question

Can multiple containers write to the same volume?

### Perfect Answer

Yes.

However, the application must be designed to handle concurrent writes safely to avoid data corruption.

---

### Question

Which storage option would you use for MySQL in production?

### Perfect Answer

I would use a named Docker volume backed by reliable storage, or an external storage solution such as Amazon EBS or Amazon EFS depending on the deployment architecture and availability requirements.

---

### Question

Can Docker volumes be moved to another server?

### Perfect Answer

Yes.

The data can be backed up and restored, or external storage systems such as NFS or EFS can be used to make data accessible across hosts.

---

# 27. Hands-on Labs

## Lab 1

Create a named volume

```bash
docker volume create app-data
```

---

## Lab 2

Run Nginx with the volume

```bash
docker run -d \
--name web \
-v app-data:/usr/share/nginx/html \
nginx
```

---

## Lab 3

Inspect the volume

```bash
docker volume inspect app-data
```

---

## Lab 4

Create a bind mount

```bash
mkdir app

docker run -d \
-v $(pwd)/app:/usr/share/nginx/html \
nginx
```

---

## Lab 5

Test persistence

1. Create a file inside the mounted directory.
2. Remove the container.
3. Start a new container with the same volume.
4. Verify the file still exists.

---

## Lab 6

List and clean up

```bash
docker volume ls

docker volume prune
```

---

# 28. One-Page Revision

```
Application

↓

Container

↓

Named Volume

↓

Host Storage
```

Remember

- Writable Layer
- Named Volume
- Bind Mount
- tmpfs
- Volume Drivers
- Backup
- Restore
- Persistent Storage

---

# 29. Think Like a Production Engineer

Don't think:

> "Volumes store files."

Think:

> "Volumes separate application lifecycle from data lifecycle."

Whenever someone says:

```
Database Lost

↓

Container Recreated

↓

Files Missing
```

Your troubleshooting flow should be:

```
docker inspect

↓

docker volume ls

↓

docker volume inspect

↓

Mount Path

↓

Permissions

↓

Application Logs
```

---

# 30. Key Takeaways

Docker volumes provide persistent storage independent of containers. Named volumes are the preferred production choice, bind mounts are ideal for development, and tmpfs is useful for temporary in-memory data. Understanding Docker storage is essential before moving to Amazon EBS, Amazon EFS, Kubernetes Persistent Volumes (PV), and Persistent Volume Claims (PVC).

# End of Chapter 03
