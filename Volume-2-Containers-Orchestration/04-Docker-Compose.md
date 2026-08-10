# Docker Compose

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 04

---

# Chapter Objective

After completing this chapter, you should be able to:

- Understand why Docker Compose is used.
- Write production-ready compose.yaml files.
- Manage multi-container applications.
- Understand services, networks, and volumes.
- Configure environment variables.
- Scale services.
- Troubleshoot Docker Compose deployments.
- Answer Docker Compose interview questions.

---

# 1. Why Docker Compose?

Suppose your application consists of:

- Spring Boot Application
- MySQL Database
- Redis Cache
- Nginx Reverse Proxy

Without Docker Compose, you would need multiple `docker run` commands.

Example:

```bash
docker run mysql ...
docker run redis ...
docker run nginx ...
docker run app ...
```

Managing dependencies becomes difficult.

Docker Compose solves this.

---

# 2. What is Docker Compose?

Docker Compose is a tool used to define and run multi-container Docker applications using a single YAML file.

Instead of remembering many commands, you describe the entire application in one file.

```
compose.yaml

↓

docker compose up

↓

Entire Application Starts
```

---

# 3. Compose Architecture

```
compose.yaml

↓

Docker Compose

↓

Docker Engine

↓

Networks

↓

Volumes

↓

Containers
```

Compose communicates with the Docker Engine API.

---

# 4. Basic compose.yaml

```yaml
services:

  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
```

Run

```bash
docker compose up
```

Stop

```bash
docker compose down
```

---

# 5. Multiple Services

Example

```yaml
services:

  app:
    image: myapp:1.0

  mysql:
    image: mysql:8

  redis:
    image: redis:7
```

One command starts all services.

---

# 6. Service

⭐⭐⭐⭐⭐

Every application component is defined as a **service**.

Example

```
Frontend

↓

Backend

↓

Database
```

Each is an independent container.

---

# 7. Container Names

Example

```yaml
services:

  app:
    container_name: app
```

Without specifying `container_name`, Docker Compose generates one automatically.

Production recommendation:

Avoid hardcoding container names unless required.

---

# 8. Port Mapping

Example

```yaml
ports:

  - "8080:80"
```

Meaning

```
Host

8080

↓

Container

80
```

---

# 9. Environment Variables

Example

```yaml
environment:

  MYSQL_ROOT_PASSWORD: password

  MYSQL_DATABASE: appdb
```

Equivalent to

```bash
docker run -e MYSQL_ROOT_PASSWORD=password
```

---

# 10. Environment File (.env)

Instead of storing values directly:

```
.env

MYSQL_PASSWORD=password

MYSQL_USER=admin
```

Compose

```yaml
env_file:

  - .env
```

Better for maintainability.

Do not commit sensitive `.env` files to Git.

---

# 11. Volumes

Example

```yaml
volumes:

  mysql-data:
```

Attach

```yaml
services:

  mysql:

    volumes:

      - mysql-data:/var/lib/mysql
```

Data survives container recreation.

---

# 12. Networks

Example

```yaml
networks:

  app-network:
```

Use

```yaml
services:

  app:

    networks:

      - app-network
```

Containers communicate using service names.

---

# 13. depends_on

Example

```yaml
depends_on:

  - mysql
```

Compose starts MySQL before the application.

Important:

`depends_on` controls startup order, **not application readiness**.

The database may still be initializing.

---

# 14. Restart Policy

Example

```yaml
restart: always
```

Other options

```
no

on-failure

unless-stopped
```

Useful for production resilience.

---

# 15. Build vs Image

Image

```yaml
image: nginx
```

Build

```yaml
build: .
```

Use `image` when the image already exists.

Use `build` when creating the image from a Dockerfile.

---

# 16. Build Context

Example

```yaml
build:

  context: .

  dockerfile: Dockerfile
```

Compose builds the image before starting the container.

---

# 17. Health Check

Example

```yaml
healthcheck:

  test: ["CMD", "curl", "-f", "http://localhost"]

  interval: 30s

  retries: 3
```

Allows Docker to determine whether a container is healthy.

---

# 18. Scaling

Example

```bash
docker compose up --scale app=3
```

Creates

```
App 1

App 2

App 3
```

Note:

Production orchestration is typically handled by ECS or Kubernetes.

---

# 19. Logs

View all logs

```bash
docker compose logs
```

Specific service

```bash
docker compose logs app
```

Follow logs

```bash
docker compose logs -f
```

---

# 20. Common Commands

Start

```bash
docker compose up
```

Detached

```bash
docker compose up -d
```

Stop

```bash
docker compose down
```

Restart

```bash
docker compose restart
```

Build

```bash
docker compose build
```

List services

```bash
docker compose ps
```

---

# 21. Sample Production Compose

```yaml
version: "3.9"

services:

  app:

    image: springboot-app:1.0

    ports:
      - "8080:8080"

    environment:
      SPRING_PROFILES_ACTIVE: prod

    depends_on:
      - mysql

  mysql:

    image: mysql:8

    environment:
      MYSQL_ROOT_PASSWORD: password

    volumes:
      - mysql-data:/var/lib/mysql

volumes:

  mysql-data:
```

---

# 22. Production Scenario 1

Problem

Application cannot connect to MySQL.

Check

- Service name
- Network
- Environment variables
- Database readiness

---

# 23. Production Scenario 2

Problem

Changes to code are not reflected.

Possible Causes

- Image not rebuilt
- Bind mount missing
- Browser cache

Solution

```bash
docker compose build

docker compose up -d
```

---

# 24. Production Scenario 3

Problem

Database data disappears.

Cause

No volume attached.

Solution

Use a named volume.

---

# 25. Production Scenario 4

Problem

Application exits immediately.

Check

```bash
docker compose logs

docker inspect
```

Possible Causes

- Missing environment variables
- Incorrect command
- Dependency failure

---

# 26. Best Practices

- Use compose.yaml instead of many docker run commands.
- Store configuration in `.env`.
- Use named volumes for databases.
- Create custom networks.
- Add health checks.
- Pin image versions.
- Keep one responsibility per service.

---

# 27. Common Mistakes

❌ Using `latest` tag.

Use explicit versions.

---

❌ Hardcoding passwords.

Use `.env` or a secrets manager.

---

❌ Assuming `depends_on` waits for readiness.

It only controls startup order.

---

❌ Running databases without volumes.

Data loss risk.

---

# 28. Interview Questions

## Question 1

What is Docker Compose?

### Perfect Answer

Docker Compose is a tool used to define and manage multi-container Docker applications using a single YAML file. It simplifies deployment by describing services, networks, volumes, and configuration in one place.

---

## Question 2

Difference between `docker run` and Docker Compose?

### Perfect Answer

`docker run` starts a single container manually.

Docker Compose manages multiple related containers declaratively using a compose file.

---

## Question 3

What is `depends_on`?

### Perfect Answer

`depends_on` controls the startup order of services. It does not guarantee that the dependent service is fully initialized or ready to accept connections.

---

## Question 4

How do containers communicate in Docker Compose?

### Perfect Answer

Services connected to the same Compose network communicate using their service names through Docker's embedded DNS.

---

## Question 5

Where would you store database credentials?

### Perfect Answer

For development, an `.env` file can be used.

For production, credentials should be stored in a secure secrets management solution such as Docker Secrets, AWS Secrets Manager, or HashiCorp Vault.

---

# 29. Amazon Cross Questions

### Question

Can Docker Compose be used in production?

### Perfect Answer

It can be used for small deployments, development, or testing. For large-scale production environments, container orchestrators such as Amazon ECS or Kubernetes are generally preferred.

---

### Question

If MySQL takes 30 seconds to start, will `depends_on` wait?

### Perfect Answer

No.

`depends_on` only controls startup order. Additional health checks or wait mechanisms are required to ensure the application starts after the database is ready.

---

### Question

Can multiple Compose projects run on the same host?

### Perfect Answer

Yes.

Docker Compose creates separate project-specific networks and resource names by default, allowing multiple projects to coexist.

---

### Question

How do you update a service to a new image version?

### Perfect Answer

Update the image tag in the compose file, pull or build the new image if required, and recreate the service using:

```bash
docker compose up -d
```

---

# 30. Hands-on Labs

## Lab 1

Create a Compose file with Nginx.

---

## Lab 2

Add a MySQL service.

---

## Lab 3

Attach a named volume to MySQL.

---

## Lab 4

Create a custom network.

---

## Lab 5

Store database credentials in a `.env` file.

---

## Lab 6

Scale the application.

```bash
docker compose up --scale app=3
```

---

## Lab 7

View logs.

```bash
docker compose logs -f
```

---

# 31. One-Page Revision

```
compose.yaml

↓

Docker Compose

↓

Docker Engine

↓

Services

↓

Networks

↓

Volumes

↓

Containers
```

Remember

- Services
- Networks
- Volumes
- Environment Variables
- depends_on
- Health Checks
- Build
- Image
- Restart Policy

---

# 32. Think Like a Production Engineer

Don't think:

> "Docker Compose starts multiple containers."

Think:

> "Docker Compose defines an application's infrastructure as code for local development and testing, making deployments repeatable and consistent."

Whenever someone says:

```
Application Not Starting

↓

Database Connection Failed

↓

Containers Cannot Communicate
```

Your troubleshooting flow should be:

```
docker compose ps

↓

docker compose logs

↓

docker network inspect

↓

Environment Variables

↓

Health Checks

↓

Application Logs
```

---

# Key Takeaways

Docker Compose simplifies multi-container application deployment by defining services, networks, volumes, and configuration in a single YAML file. It is an excellent tool for development, testing, and smaller deployments, while also providing concepts that translate directly to ECS task definitions and Kubernetes manifests.

# End of Chapter 04
