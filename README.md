The goal is to turn your existing Docker repository into a **hands-on Docker command center for interview preparation, troubleshooting, and real-world DevOps work**. Since you already have the theory, this cheat sheet will focus on commands you can execute, configurations you can write, failures you can reproduce, and problems you can troubleshoot.

I'll structure it so you can use the material in three ways:
1. **Quick reference:** Commands, syntax, flags, and common patterns.
2. **Hands-on lab manual:** Exercises you can perform on your machine.
3. **Advanced practice:** Multi-stage builds, distroless images, Buildx, volumes, networking, security, and troubleshooting.

# Docker Cheat Sheet: Beginner to Advanced

**Recommended repository:** `Docker-Notes`  
**Objective:** Become confident building, running, securing, optimizing, and troubleshooting Docker containers.

**Environment:** Docker Desktop on Windows with WSL 2, or Docker Engine on Linux. Most commands below work on both. Use PowerShell or Windows Terminal on Windows, and Bash on Linux.

**Your working rule:** Don't just read a command. Execute it, inspect its output, break something intentionally, and fix it.

## 1. Your Docker command reference

Official reference: [Docker CLI documentation](https://docs.docker.com/reference?utm_source=chatgpt.com)

### 1.1 Check Docker installation and environment

```bash
docker --version
docker version
docker info
docker system df
docker context ls
docker context show
docker buildx version
docker compose version
```

| Command | What you learn |
|---|---|
| `docker --version` | Installed Docker CLI version |
| `docker version` | Client and server/engine versions |
| `docker info` | Engine configuration, storage driver, resources and runtime information |
| `docker system df` | Disk usage by images, containers, volumes and build cache |
| `docker context ls` | Available Docker environments |
| `docker context show` | Currently selected context |
| `docker buildx version` | Buildx availability |
| `docker compose version` | Compose availability |

**Hands-on:** Run `docker info`. Identify the storage driver, container count, image count, and number of CPUs reported to Docker.

### 1.2 Image commands

```bash
# List local images
docker images
docker image ls

# Download an image
docker pull nginx:alpine

# Inspect image details
docker image inspect nginx:alpine

# View image history and layers
docker history nginx:alpine

# Tag an image
docker tag nginx:alpine my-nginx:v1

# Remove an image
docker image rm my-nginx:v1

# Remove unused images
docker image prune

# Remove all unused images, including unreferenced tagged images
docker image prune -a
```

**Interview distinction:**
- An **image** is a packaged template.
- A **container** is a running or stopped instance created from an image.
- A **registry** stores and distributes images.
- A **layer** is a filesystem change recorded during image construction.

### 1.3 Container lifecycle commands

```bash
# Create and start a container
docker run --name web nginx:alpine

# Start in detached mode
docker run -d --name web -p 8080:80 nginx:alpine

# List running containers
docker ps

# List all containers, including stopped ones
docker ps -a

# Stop and start
docker stop web
docker start web
docker restart web

# Stop gracefully, then force-remove if necessary
docker stop web
docker rm web

# Follow container logs
docker logs -f web

# Show the last 100 log lines
docker logs --tail 100 web

# Display live resource usage
docker stats

# Show container configuration and state
docker inspect web

# Execute a command inside a running container
docker exec -it web sh

# Display the container's processes
docker top web

# Copy files between host and container
docker cp web:/etc/nginx/nginx.conf ./nginx.conf

# Display port mappings
docker port web

# Show filesystem changes relative to the image
docker diff web
```

**Important:** `docker run` creates a new container. `docker start` starts an existing stopped container.

`docker exec` runs a new process inside a running container. `docker attach` connects to its primary process's standard input/output and can behave differently when you send signals.

### 1.4 Run containers with useful options

```bash
# Run in background and publish a host port
docker run -d --name web -p 8080:80 nginx:alpine

# Assign a restart policy
docker run -d --restart unless-stopped nginx:alpine

# Set environment variables
docker run -d -e APP_ENV=development nginx:alpine

# Set a container hostname
docker run -d --hostname app01 nginx:alpine

# Limit memory and CPU
docker run -d --memory=512m --cpus=1 nginx:alpine

# Remove the container automatically after it exits
docker run --rm alpine:3.22 echo "Hello Docker"

# Run interactively
docker run --rm -it alpine:3.22 sh

# Set a read-only root filesystem
docker run -d --read-only --tmpfs /tmp nginx:alpine

# Set a user
docker run -d --user 10001:10001 nginx:alpine
```

**Remember:** A container's configured hostname is not the same thing as a Docker network's service-discovery name.

### 1.5 Registry commands

```bash
# Authenticate to Docker Hub
docker login

# Tag a local image for your Docker Hub repository
docker tag myapp:v1 YOUR_DOCKERHUB_USERNAME/myapp:v1

# Upload the image
docker push YOUR_DOCKERHUB_USERNAME/myapp:v1

# Download an image
docker pull YOUR_DOCKERHUB_USERNAME/myapp:v1

# Log out
docker logout
```

Never put passwords, access tokens, or cloud credentials directly into a Dockerfile or commit them to Git.

---

## 2. Write your first Dockerfile from scratch

Official tutorial: [Writing a Dockerfile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile?utm_source=chatgpt.com)

### Lab 01 — Build and run your own web server

**Objective:** Understand the complete path from source files to image to running container.

Create this folder structure:

```text
docker-lab-01/
├── Dockerfile
├── index.html
└── .dockerignore
```

**File 1: `index.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My Docker Lab</title>
</head>
<body>
    <h1>Hello from my Docker container!</h1>
    <p>Built from my own Dockerfile.</p>
</body>
</html>
```

**File 2: `Dockerfile`**

```dockerfile
FROM nginx:alpine

LABEL maintainer="docker-lab"
LABEL purpose="Docker hands-on practice"

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**File 3: `.dockerignore`**

```text
.git
.gitignore
.env
*.log
node_modules
```

The `.dockerignore` file prevents matching files from being sent as part of the build context. It is especially important for excluding local dependencies, secrets, and unnecessary files.

### Build the image

Run these commands from inside `docker-lab-01`:

```bash
docker build -t my-docker-web:v1 .
docker image ls
docker image inspect my-docker-web:v1
docker history my-docker-web:v1
```

### Run the container

```bash
docker run -d \
  --name my-web \
  -p 8080:80 \
  my-docker-web:v1
```

On Windows PowerShell, the equivalent single-line command is:

```powershell
docker run -d --name my-web -p 8080:80 my-docker-web:v1
```

Open `http://localhost:8080` in your browser.

### Verify everything

```bash
docker ps
docker logs my-web
docker port my-web
docker inspect my-web
docker stats my-web
```

### Break it and fix it

Perform these exercises:

1. Change the heading in `index.html`. Rebuild the image and recreate the container. Does the old container automatically receive your changes?
2. Try to start another container using host port `8080`. What error appears?
3. Stop the container and run `docker ps`. Then run `docker ps -a`. What is the difference?
4. Remove the container, but keep the image. Can you create a new container without rebuilding?
5. Change the host port to `8081` while keeping the container port at `80`. Why does the application still listen on port `80` inside the container?

**Success criteria:** You can explain the difference between building an image, creating a container, starting a container, and publishing a port without looking at your notes.

---

## 3. Dockerfile instructions you must master

| Instruction | Purpose | Practical example |
|---|---|---|
| `FROM` | Select a base image or begin a new build stage | `FROM python:3.13-slim` |
| `WORKDIR` | Set the working directory | `WORKDIR /app` |
| `COPY` | Copy files into an image | `COPY . .` |
| `ADD` | Copy files, with additional archive/URL functionality | Prefer `COPY` for ordinary local files |
| `RUN` | Execute a command during the build | `RUN pip install -r requirements.txt` |
| `CMD` | Set the default command and/or arguments | `CMD ["python", "app.py"]` |
| `ENTRYPOINT` | Define the main executable | `ENTRYPOINT ["python", "app.py"]` |
| `ENV` | Set environment variables in the image | `ENV PYTHONUNBUFFERED=1` |
| `ARG` | Declare build-time variables | `ARG APP_VERSION=1.0` |
| `EXPOSE` | Document the intended container port | `EXPOSE 8080` |
| `USER` | Set the user for subsequent instructions and runtime | `USER 10001:10001` |
| `LABEL` | Add image metadata | `LABEL app="demo"` |
| `HEALTHCHECK` | Define a container health-check command | `HEALTHCHECK CMD ...` |
| `VOLUME` | Declare a mount point | `VOLUME ["/data"]` |
| `STOPSIGNAL` | Configure the signal used to stop the container | `STOPSIGNAL SIGTERM` |

Full reference: [Dockerfile instruction reference](https://docs.docker.com/reference/dockerfile?utm_source=chatgpt.com)

### RUN vs CMD vs ENTRYPOINT

This is a common interview question.

**RUN:** Executes while building the image.

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

**CMD:** Supplies the default command or arguments when a container starts. It can be overridden.

```dockerfile
CMD ["python", "app.py"]
```

**ENTRYPOINT:** Defines the executable that the container runs by default.

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

You can combine them:

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]
```

The default execution becomes:

```bash
python app.py --port 8080
```

Overriding the command at runtime can replace the default `CMD` arguments.

**Hands-on challenge:** Create a small Dockerfile with `ENTRYPOINT ["echo"]` and `CMD ["Hello Docker"]`. Run the image normally, then override the command with `docker run IMAGE "Hello DevOps"`. Observe the difference.

### Shell form vs exec form

```dockerfile
# Shell form
CMD python app.py

# Exec form
CMD ["python", "app.py"]
```

Prefer the exec form for application processes. It avoids an unnecessary shell wrapper and helps the application receive container stop signals correctly.

---

## 4. Build optimization: Layers, caching, and image size

The first step toward becoming good at Docker is understanding why a build is slow or an image is unnecessarily large.

### 4.1 Understand layer caching

Consider this inefficient Dockerfile:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY . .

RUN pip install --no-cache-dir -r requirements.txt

CMD ["python", "app.py"]
```

Whenever an application source file changes, the `COPY . .` instruction can invalidate the cache for the subsequent dependency installation.

A better approach:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Dependencies are installed before frequently changing application code is copied.

### 4.2 Build and measure

```bash
docker build -t python-demo:v1 .
docker image ls
docker history python-demo:v1
```

Build again without changing any files. Observe the cached steps.

Now change `app.py` and rebuild:

```bash
docker build -t python-demo:v2 .
```

Observe which steps execute again.

### 4.3 Commands for build performance

```bash
# Build using BuildKit through Buildx
docker buildx build -t myapp:dev .

# Show build output in a more detailed format
docker buildx build --progress=plain -t myapp:dev .

# Build without using cached layers
docker build --no-cache -t myapp:dev .

# Remove unused build cache
docker builder prune

# Display Buildx builders
docker buildx ls

# Inspect the selected builder
docker buildx inspect

# Inspect a named builder
docker buildx inspect mybuilder
```

**Important distinction:** `--no-cache` disables reuse of build cache; it does not automatically pull a fresh version of every base image. Use `--pull` when you also want to check for a newer base image.

```bash
docker build --pull --no-cache -t myapp:dev .
```

### 4.4 Build a clean image

For Debian-based images, combine package installation and cleanup in the same `RUN` instruction:

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
```

For Alpine-based images:

```dockerfile
RUN apk add --no-cache curl
```

Do not install troubleshooting tools, compilers, or package managers in the final production image unless they are genuinely needed.

**Practice task:** Build the same application three times:
1. With an inefficient Dockerfile.
2. With improved layer ordering.
3. With a multi-stage Dockerfile.

Record build duration, image size, and what changes between builds.

---

## 5. Multi-stage Docker builds

Official guide: [Docker multi-stage builds](https://docs.docker.com/get-started/docker-concepts/building-images/multi-stage-builds/?utm_source=chatgpt.com)

Multi-stage builds let you use one environment to build an application and a different environment to run it.

**Why it matters:**
- Smaller production images.
- Fewer unnecessary tools in production.
- Reduced attack surface.
- Separation of build-time and runtime dependencies.

### Lab 02 — Build a Java application using multiple stages

Use a simple Java program.

Folder structure:

```text
java-multistage/
├── Dockerfile
└── Main.java
```

`Main.java`:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello from a multi-stage Docker build!");
    }
}
```

`Dockerfile`:

```dockerfile
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /app

COPY Main.java .

RUN javac Main.java


FROM eclipse-temurin:21-jre AS runtime

WORKDIR /app

COPY --from=builder /app/Main.class .

USER 10001:10001

CMD ["java", "Main"]
```

### Build and run

```bash
docker build -t java-multistage:v1 .
docker run --rm java-multistage:v1
docker image ls
docker history java-multistage:v1
```

Expected application output:

```text
Hello from a multi-stage Docker build!
```

### Understand the important lines

`FROM eclipse-temurin:21-jdk AS builder` creates a stage with the Java compiler.

`RUN javac Main.java` compiles the program.

`FROM eclipse-temurin:21-jre AS runtime` starts a separate final stage with the Java runtime.

`COPY --from=builder /app/Main.class .` copies the compiled application from the builder stage.

The final image doesn't need the JDK or source file.

### Advanced multi-stage exercises

1. Add a third stage called `test` that runs a test before creating the final image.
2. Create a `debug` target that includes troubleshooting tools.
3. Build only the builder stage using `--target builder`.
4. Compare the JDK-based image with the JRE-based image.
5. Change the application source and observe how Docker's cache behaves.

Build a particular stage:

```bash
docker build --target builder -t java-builder:dev .
```

**Interview question:** If a file is created in one stage, why doesn't it automatically appear in the next stage?

Because each `FROM` begins a new stage. Files and build state are not inherited from the previous stage unless you use the previous stage as a base or explicitly copy what you need.

---

## 6. Distroless images: Production hardening

A distroless image contains an application and its runtime dependencies without the usual operating-system shell and package-management utilities.

Google's official project: [GoogleContainerTools Distroless](https://github.com/GoogleContainerTools/distroless?utm_source=chatgpt.com)

### Lab 03 — Build a Java application with a distroless runtime

Use the same `Main.java` from Lab 02.

`Dockerfile.distroless`:

```dockerfile
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /app

COPY Main.java .

RUN javac Main.java


FROM gcr.io/distroless/java21-debian13:nonroot AS runtime

WORKDIR /app

COPY --from=builder --chown=nonroot:nonroot /app/Main.class .

CMD ["Main"]
```

Build and run:

```bash
docker build -f Dockerfile.distroless -t java-distroless:v1 .
docker run --rm java-distroless:v1
```

The runtime image uses a Java runtime but does not provide the usual interactive shell.

**Note:** Distroless image availability and tags can change. If the selected image tag isn't available, check the project's official supported image names and tags.

### Debugging distroless containers

This is where you need to think like a production engineer.

A normal container might allow:

```bash
docker exec -it myapp sh
```

A distroless container generally won't have `sh`, `bash`, `curl`, or `apt`. The command above can fail even when the application is running correctly.

Use these methods instead:

```bash
docker ps -a
docker logs myapp
docker inspect myapp
docker top myapp
docker stats myapp
docker port myapp
```

You can also run a separate debugging container on the same Docker network or use supported ephemeral-debugging techniques in your orchestrator.

### Distroless challenge

Create two images from the same Java program:

- Image A: Java runtime based on a conventional Linux distribution.
- Image B: Java runtime using a distroless image.

Compare:

| Test | Conventional runtime | Distroless runtime |
|---|---|---|
| Does the application start? | Test it | Test it |
| Can you run `sh` inside it? | Depends on the image | Normally no |
| Can you run `apt` inside it? | Depends on the image | No |
| Can you read container logs? | Yes, through Docker | Yes, through Docker |
| Does it contain the JDK? | Depends on the selected image | Not in this example |
| Image size | Measure it | Measure it |

**Do not assume distroless automatically means secure.** You still need patched base images, trusted dependencies, correct permissions, vulnerability scanning, and a secure runtime configuration.

---

## 7. Docker volumes and bind mounts

Official guide: [Docker storage documentation](https://docs.docker.com/engine/storage/?utm_source=chatgpt.com)

Containers are replaceable. Data that must survive container replacement should be stored outside the container's writable layer.

There are three important storage concepts:

| Storage type | Data location | Best use |
|---|---|---|
| Named volume | Managed by Docker | Persistent application or database data |
| Bind mount | A specific host path | Development, configuration files, local source code |
| tmpfs mount | Memory-backed filesystem on supported Linux configurations | Temporary, nonpersistent files |

### 7.1 Named volume

Create a volume:

```bash
docker volume create app-data
docker volume ls
docker volume inspect app-data
```

Run a container using the volume:

```bash
docker run -d \
  --name volume-demo \
  --mount type=volume,src=app-data,dst=/data \
  alpine:3.22 \
  sh -c 'echo "Persistent Docker data" > /data/message.txt; sleep 3600'
```

On PowerShell, use the same command on one line if multiline shell syntax causes issues.

Read the file:

```bash
docker exec volume-demo cat /data/message.txt
```

Now remove the container:

```bash
docker rm -f volume-demo
```

Create another container using the same volume:

```bash
docker run --rm \
  --mount type=volume,src=app-data,dst=/data \
  alpine:3.22 \
  cat /data/message.txt
```

Expected output:

```text
Persistent Docker data
```

The volume persists after the first container is removed.

Clean up when finished:

```bash
docker volume rm app-data
```

### 7.2 Bind mounts

A bind mount maps a specific host file or directory into a container.

Create a host directory called `site` and put an `index.html` file inside it.

```bash
docker run -d \
  --name bind-demo \
  -p 8080:80 \
  --mount type=bind,src=./site,dst=/usr/share/nginx/html,readonly \
  nginx:alpine
```

If your shell or Docker environment requires an absolute host path, use the full path to `site`.

Check the mount:

```bash
docker inspect bind-demo
```

Update the host's `index.html`, refresh the browser, and observe the change.

The `readonly` option prevents the container from writing to the mounted directory.

**Windows note:** Docker Desktop handles host paths through its Linux VM/WSL integration. If a mount fails, verify the path, shared filesystem access, and the shell's path syntax.

### 7.3 Volume vs bind mount: interview answer

- Use **named volumes** for data that Docker should manage independently of the host's project directory.
- Use **bind mounts** when you need to access a specific host file or directory.
- Use **tmpfs** when data should be temporary and memory-backed on a supported Linux environment.

### 7.4 Volume troubleshooting

```bash
docker volume ls
docker volume inspect app-data
docker inspect volume-demo
docker logs volume-demo
docker exec volume-demo ls -la /data
```

Common mistakes:
- Mounting to the wrong container path.
- Accidentally hiding files already present at the mount destination.
- Running the application as a user without the necessary permissions.
- Deleting a volume when intending to delete only a container.
- Assuming a bind mount has the same behavior across Windows and Linux.

**Hands-on challenge:** Run a database with a named volume, create a record, replace the container, and verify that the record remains. Only remove the volume when you deliberately want to delete the stored data.

---

## 8. Docker networking

Official guide: [Docker networking documentation](https://docs.docker.com/engine/network/?utm_source=chatgpt.com)

You should understand how containers communicate with one another, how ports are published, and why `localhost` often causes problems.

### 8.1 List and inspect networks

```bash
docker network ls
docker network inspect bridge
docker network inspect host
docker network inspect none
```

Common network drivers:

| Driver | Purpose |
|---|---|
| `bridge` | Connect containers on a Docker host |
| `host` | Use the host network directly, where supported |
| `none` | Disable external container networking |
| `overlay` | Connect services across hosts in supported Swarm environments |
| `macvlan` | Give containers network identities on a physical network |

For ordinary single-host development, a **user-defined bridge network** is a great place to start.

### 8.2 Lab 04 — Build your own network

Create a network:

```bash
docker network create app-network
```

Start two containers on that network:

```bash
docker run -d --name web1 --network app-network nginx:alpine
docker run -d --name web2 --network app-network nginx:alpine
```

Check the network:

```bash
docker network inspect app-network
```

From `web1`, try reaching `web2` using its container name:

```bash
docker exec web1 wget -qO- http://web2
```

You should receive the default NGINX HTML response.

Test the reverse direction:

```bash
docker exec web2 wget -qO- http://web1
```

Why does this work? Containers attached to the same user-defined bridge network can use Docker's embedded DNS to resolve one another by name.

Remove the lab:

```bash
docker rm -f web1 web2
docker network rm app-network
```

### 8.3 Understand port mapping

```bash
docker run -d --name web -p 8080:80 nginx:alpine
```

The format is:

```text
-p HOST_PORT:CONTAINER_PORT
```

In this example:

- Host port: `8080`
- Container port: `80`
- Browser URL: `http://localhost:8080`

Without `-p`, a port is not automatically published to the host just because the Dockerfile contains `EXPOSE`.

To publish a port only on the host's loopback interface:

```bash
docker run -d --name private-web -p 127.0.0.1:8081:80 nginx:alpine
```

This is useful when a service should be reachable locally but not directly exposed on all host interfaces.

### 8.4 The localhost problem

Imagine you have two containers:

```text
Application container
        |
        | HTTP request
        v
Database container
```

If the application uses `localhost:5432` to reach PostgreSQL inside a different container, it attempts to connect to itself—not to the database container.

Use the database's network alias or service name instead, such as:

```text
postgres:5432
```

Inside a container, `localhost` refers to that container's own network namespace, unless a special networking configuration changes the situation.

### 8.5 Troubleshoot Docker networking

```bash
docker network ls
docker network inspect app-network
docker inspect web1
docker port web1
docker logs web1
```

If the application image has the required utilities, you can test connectivity from inside it:

```bash
docker exec web1 wget -qO- http://web2
```

For more complete troubleshooting, run a temporary diagnostic container on the same network:

```bash
docker run --rm -it --network app-network alpine:3.22 sh
```

Inside the diagnostic container:

```sh
# Install diagnostic utilities if needed
apk add --no-cache curl bind-tools busybox-extras

# Resolve the other container
nslookup web2

# Test HTTP connectivity
curl -v http://web2

# Test a TCP port
nc -zv web2 80
```

These tools are for troubleshooting; you don't need to install them in every production image.

**Practice:** Break your application by removing one container from the shared network. Diagnose the failed request, reconnect the container, and verify recovery.

---

## 9. Docker Buildx: Multi-platform builds and advanced image building

Official guide: [Docker Buildx documentation](https://docs.docker.com/build/buildx/?utm_source=chatgpt.com)

Buildx provides advanced build capabilities, including multi-platform builds, remote builders, build cache management, and publishing images directly to registries.

### 9.1 Inspect available builders

```bash
docker buildx version
docker buildx ls
docker buildx inspect
```

Create a builder:

```bash
docker buildx create --name mybuilder --driver docker-container --use
docker buildx inspect --bootstrap
```

The `--bootstrap` option initializes the builder and helps check that it is ready.

### 9.2 Build a local image

```bash
docker buildx build --load -t myapp:dev .
```

`--load` loads the built image into the local Docker image store when supported by the selected builder and output format.

Check the image:

```bash
docker image ls
docker image inspect myapp:dev
```

### 9.3 Build for multiple architectures

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t YOUR_DOCKERHUB_USERNAME/myapp:v1 \
  --push .
```

Replace the example username and image name with your own.

This command builds for both AMD64 and ARM64 and publishes the multi-platform image to a registry.

**Important:** A multi-platform build cannot generally load a multi-platform manifest into the classic local image store as though it were one ordinary platform-specific image. Use `--push` for publishing a multi-platform image, or use an appropriate container-image output for your builder.

### 9.4 Inspect the image platforms

```bash
docker buildx imagetools inspect YOUR_DOCKERHUB_USERNAME/myapp:v1
```

This helps verify which platform variants are available in the registry.

### 9.5 Build caching

```bash
docker buildx build \
  --cache-from type=registry,ref=YOUR_DOCKERHUB_USERNAME/myapp:buildcache \
  --cache-to type=registry,ref=YOUR_DOCKERHUB_USERNAME/myapp:buildcache,mode=max \
  -t YOUR_DOCKERHUB_USERNAME/myapp:v1 \
  --push .
```

This illustrates registry-based cache import and export. In real CI/CD, make sure the cache reference, credentials, and concurrent build strategy are configured correctly.

### 9.6 Buildx practical exercises

1. Build your application locally with `--load`.
2. Create a second builder and inspect it.
3. Build for `linux/amd64`.
4. Build for `linux/arm64`.
5. Publish a multi-platform image to Docker Hub.
6. Inspect the manifest and verify the available architectures.
7. Build twice with registry caching and compare the results.

**Interview question:** What is the difference between an image and a multi-platform manifest?

An image is the package used to run a container on a particular platform. A multi-platform manifest or image index points to the appropriate platform-specific image manifests, allowing a compatible client to select the correct image for its architecture.

---

## 10. Troubleshooting commands: Your production toolkit

Official references: [Docker CLI documentation](https://docs.docker.com/reference?utm_source=chatgpt.com) and [Docker networking documentation](https://docs.docker.com/engine/network/?utm_source=chatgpt.com)

When an application fails, don't immediately rebuild the image or restart everything. Identify the layer where the failure occurs.

### 10.1 Container troubleshooting

```bash
# Is the container running?
docker ps -a

# What happened during startup?
docker logs --tail 200 CONTAINER_NAME

# Follow new log messages
docker logs -f CONTAINER_NAME

# What is its current state and exit code?
docker inspect CONTAINER_NAME

# Which processes are running?
docker top CONTAINER_NAME

# What resources is it consuming?
docker stats

# Which ports are published?
docker port CONTAINER_NAME

# What changed in its writable filesystem?
docker diff CONTAINER_NAME
```

To print a container's state and exit code:

```bash
docker inspect --format='{{.State.Status}} {{.State.ExitCode}}' CONTAINER_NAME
```

To print its restart count:

```bash
docker inspect --format='{{.RestartCount}}' CONTAINER_NAME
```

These Go-template formats are useful in Linux shells and PowerShell.

### 10.2 Image troubleshooting

```bash
docker image ls
docker image inspect IMAGE_NAME
docker history IMAGE_NAME
docker image inspect --format='{{.Config.User}}' IMAGE_NAME
docker image inspect --format='{{json .Config.ExposedPorts}}' IMAGE_NAME
docker image inspect --format='{{json .Config.Env}}' IMAGE_NAME
```

Remember that image configuration may contain environment variables you didn't intend to expose. Never bake secrets into an image.

### 10.3 Build troubleshooting

```bash
docker build --progress=plain -t myapp:debug .
docker build --no-cache -t myapp:debug .
docker build --pull -t myapp:debug .
docker buildx ls
docker buildx inspect
docker system df
docker builder prune
```

Use `--no-cache` selectively. If a build fails because a dependency server is unavailable, rebuilding without cache may make the problem worse rather than better.

### 10.4 Docker daemon troubleshooting

```bash
docker version
docker info
docker context show
docker context ls
```

On Linux systems using systemd:

```bash
sudo systemctl status docker
sudo journalctl -u docker -n 100 --no-pager
```

If you use Docker Desktop on Windows, check whether Docker Desktop is running and whether the expected Linux or Windows container engine is selected. The Linux `systemctl` commands above are not native Windows commands.

### 10.5 Disk and cleanup troubleshooting

```bash
docker system df
docker system df -v

# Remove stopped containers
docker container prune

# Remove unused networks
docker network prune

# Remove unused images
docker image prune

# Remove unused build cache
docker builder prune
```

Be cautious with destructive commands:

```bash
docker system prune -a
docker system prune -a --volumes
```

The first removes unused containers, networks, images, and build cache according to the command's rules. The second also includes eligible unused anonymous volumes.

**Do not run cleanup commands blindly on a production machine.** Review the resources first, especially volumes containing persistent data.

---

## 11. Docker Compose: Run a complete application stack

Official guide: [Docker Compose documentation](https://docs.docker.com/compose/?utm_source=chatgpt.com)

Real applications rarely run in a single container. You may need an application, database, cache, and reverse proxy.

Docker Compose lets you define and run this multi-container environment using a YAML file.

### Lab 05 — Application + PostgreSQL

Create this directory:

```text
compose-lab/
├── compose.yaml
└── app/
    ├── Dockerfile
    ├── app.py
    └── requirements.txt
```

For this exercise, use a small Flask application.

`app/requirements.txt`:

```text
Flask
psycopg[binary]
```

`app/app.py`:

```python
import os
import time

import psycopg
from flask import Flask, jsonify

app = Flask(__name__)

def get_database_status():
    try:
        with psycopg.connect(
            host=os.getenv("DB_HOST", "db"),
            port=int(os.getenv("DB_PORT", "5432")),
            dbname=os.getenv("POSTGRES_DB", "appdb"),
            user=os.getenv("POSTGRES_USER", "appuser"),
            password=os.environ["POSTGRES_PASSWORD"],
            connect_timeout=3,
        ) as connection:
            with connection.cursor() as cursor:
                cursor.execute("SELECT 1")
                return "connected" if cursor.fetchone()[0] == 1 else "error"
    except Exception:
        return "disconnected"

@app.get("/")
def home():
    return jsonify(
        application="docker-compose-lab",
        database=get_database_status()
    )

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`app/Dockerfile`:

```dockerfile
FROM python:3.13-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

RUN useradd --no-create-home --uid 10001 appuser

USER 10001:10001

EXPOSE 5000

CMD ["python", "app.py"]
```

For a production application, use pinned dependency versions and an appropriate production WSGI server rather than Flask's development server.

### Create `compose.yaml`

```yaml
services:
  app:
    build:
      context: ./app
    ports:
      - "127.0.0.1:5000:5000"
    environment:
      DB_HOST: db
      DB_PORT: "5432"
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: "${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD}"
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend

  db:
    image: postgres:17
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: "${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD}"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 3s
      retries: 10
    networks:
      - backend

networks:
  backend:

volumes:
  postgres-data:
```

This lab deliberately uses a PostgreSQL image with a named volume. For a production deployment, pin a suitable image version or digest, protect database credentials with a secret-management mechanism, and plan backups and upgrades.

### Set the password

In Bash:

```bash
export POSTGRES_PASSWORD='replace-with-a-strong-lab-password'
```

In PowerShell:

```powershell
$env:POSTGRES_PASSWORD = "replace-with-a-strong-lab-password"
```

Use a disposable lab password; never commit it to Git.

### Start the stack

```bash
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f
```

Open `http://localhost:5000`.

Expected response:

```json
{
  "application": "docker-compose-lab",
  "database": "connected"
}
```

### Troubleshooting exercises

```bash
# List services and their state
docker compose ps

# Inspect application logs
docker compose logs app

# Inspect database logs
docker compose logs db

# Rebuild the application
docker compose build app

# Restart the application
docker compose restart app

# Stop the stack without deleting its persistent volume
docker compose down
```

Start it again:

```bash
docker compose up -d
```

The database should retain its data because the named volume is not removed by ordinary `docker compose down`.

**Destructive warning:** `docker compose down -v` removes the Compose-managed volumes. Use it only when you intentionally want to delete the lab's persistent database data.

### Advanced Compose challenges

1. Add a Redis container and connect it to the application.
2. Create a separate frontend and backend network.
3. Configure a database health check and application retry behavior.
4. Add resource limits appropriate to your Docker environment.
5. Configure logging and inspect the resulting container logs.
6. Use a Compose development override for live source-code changes.
7. Move sensitive configuration out of committed YAML files.

---

## 12. Docker security checklist

A Docker engineer should understand how to run containers safely, not merely how to start them.

### 12.1 Run as a non-root user

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY --chown=10001:10001 app.py .

USER 10001:10001

CMD ["python", "app.py"]
```

This example assumes the application file is readable and the application doesn't need to write to a protected location.

### 12.2 Run with reduced privileges

```bash
docker run -d \
  --name secure-nginx \
  --cap-drop=ALL \
  --security-opt=no-new-privileges:true \
  -p 8080:80 \
  nginx:alpine
```

Test this configuration instead of assuming it works for every application. Some software needs additional Linux capabilities or writable directories.

### 12.3 Inspect container security configuration

```bash
docker inspect --format='{{.HostConfig.Privileged}}' CONTAINER_NAME
docker inspect --format='{{.HostConfig.ReadonlyRootfs}}' CONTAINER_NAME
docker inspect --format='{{.Config.User}}' CONTAINER_NAME
docker inspect --format='{{json .HostConfig.CapDrop}}' CONTAINER_NAME
```

### 12.4 Security best practices

- Prefer trusted, maintained base images.
- Pin image versions; use digests when reproducibility is required.
- Run applications as non-root users where possible.
- Avoid `--privileged` unless you have a specific, justified need.
- Don't mount the Docker socket into an application container casually.
- Use `.dockerignore` to exclude secrets and unnecessary files.
- Use BuildKit secret mounts for build-time credentials.
- Scan images for vulnerabilities.
- Keep the final image free of unnecessary build tools.
- Apply CPU and memory limits where appropriate.
- Use read-only filesystems where the application supports them.
- Publish only the ports that must be accessible.
- Maintain an image-update and vulnerability-remediation process.

### 12.5 BuildKit secrets: A useful advanced pattern

Suppose your build needs a token to download a private dependency.

Avoid this:

```dockerfile
ARG TOKEN
RUN download-private-dependency --token "$TOKEN"
```

Build arguments are not a suitable mechanism for secrets; values can leak through build metadata, logs, or other artifacts.

Use a secret mount instead:

```dockerfile
# syntax=docker/dockerfile:1

FROM alpine:3.22

RUN --mount=type=secret,id=api_token \
    TOKEN="$(cat /run/secrets/api_token)" && \
    test -n "$TOKEN"
```

This only demonstrates secure access to a build secret. In a real build, replace the `test` command with the required download operation and ensure the token isn't printed or written to an image layer.

For a disposable Linux or WSL lab, create a secret file outside your Git repository and run:

```bash
docker buildx build \
  --secret id=api_token,src=./api_token.txt \
  --load \
  -t secret-demo:dev .
```

Ensure `api_token.txt` is excluded from version control. Don't use a real production credential for this exercise.

---

## 13. Practice writing Dockerfiles: 10 challenges

This section is the heart of your hands-on preparation. For each challenge, write the Dockerfile yourself before looking at reference examples.

| Lab | Challenge | Skills you practise |
|---|---|---|
| 01 | Serve a static HTML page using NGINX | `FROM`, `COPY`, `EXPOSE`, `CMD` |
| 02 | Containerize a Python application | Dependencies, working directories, environment variables |
| 03 | Containerize a Node.js application | `npm`, dependency caching, startup commands |
| 04 | Containerize a Java application | JDK, JRE, compiled artifacts |
| 05 | Build a multi-stage Java image | Named stages, `COPY --from` |
| 06 | Create a distroless Java image | Minimal runtime, non-root execution, debugging |
| 07 | Run a database with persistent storage | Volumes, environment variables, data persistence |
| 08 | Build an application + database stack | Compose, networking, service discovery |
| 09 | Build an image for multiple CPU architectures | Buildx, manifests, registries |
| 10 | Harden an application container | Non-root users, capabilities, read-only filesystems |

### Your Dockerfile assessment checklist

After completing each lab, ask yourself:

- Does the image build successfully?
- Does the container start correctly?
- Does the application listen on the correct interface and port?
- Can I inspect logs and determine why it failed?
- Are dependencies cached efficiently?
- Can I reduce the image size without breaking the application?
- Is the container running as a non-root user?
- Does persistent data survive container replacement?
- Are secrets excluded from the image and repository?
- Can I explain every Dockerfile instruction in an interview?

### Bonus: Build an intentionally broken image

Create these failures one at a time:

1. Use an invalid base-image tag.
2. Copy a file that doesn't exist.
3. Specify a command that isn't installed.
4. Start an application on `127.0.0.1` inside the container when external container access is required.
5. Publish the wrong host-to-container port mapping.
6. Run as a user who cannot read the application file.
7. Connect two containers that aren't on a common network.
8. Mount an empty volume over a directory containing application files.
9. Configure a health check with an unavailable executable.
10. Make the application exit immediately after startup.

For each failure, document the symptom, the command used to investigate it, the root cause, the fix, and the verification step.

This exercise develops troubleshooting skills much faster than memorizing commands alone.

---

## 14. Common Docker problems and how to solve them

| Problem | Likely cause | First checks |
|---|---|---|
| Container exits immediately | Application exited, crashed, or command failed | `docker ps -a`, `docker logs` |
| `port is already allocated` | Another process or container uses the host port | `docker ps`, change the host port |
| `COPY failed` | Wrong path or file missing from build context | Check `pwd`, directory structure, `.dockerignore` |
| `permission denied` | Incorrect ownership, permissions, or security restrictions | `docker logs`, `docker inspect`, check `USER` |
| Application can't reach database | Wrong hostname, port, network, credentials, or database readiness | Inspect both containers and their network |
| Container cannot resolve another container | Different networks or incorrect service name | `docker network inspect`, test DNS |
| Image is too large | Unnecessary dependencies, build tools, or files | `docker history`, multi-stage build |
| Build unexpectedly reruns dependencies | Cache invalidation due to instruction ordering or changed inputs | Inspect build output and Dockerfile |
| Data disappears after container removal | Data was stored only in the container's writable layer | Inspect mounts and volumes |
| Container keeps restarting | Startup crash or restart policy | `docker inspect`, `docker logs` |
| Health check reports unhealthy | Incorrect test, unavailable tool, or failing application | `docker inspect`, check health status and logs |
| Image works on one machine but not another | Architecture, configuration, or runtime differences | `docker version`, `docker image inspect`, Buildx manifest inspection |
| Distroless container cannot open a shell | Shell is absent by design | Use logs, inspect, and a separate debugging container |
| Docker disk usage grows | Unused images, containers, volumes, or build cache | `docker system df`, inspect before pruning |

**Professional troubleshooting sequence:**

1. Confirm the exact symptom.
2. Check the container state.
3. Read the logs.
4. Inspect the configuration.
5. Verify ports and networking.
6. Check resource usage and permissions.
7. Reproduce the problem with the smallest possible setup.
8. Apply one fix at a time.
9. Verify recovery.
10. Record the root cause and preventive action.

---

## 15. The 30-day Docker mastery plan

You already have the theory. Follow this plan by **writing commands and Dockerfiles from memory**, rather than spending most of your time watching tutorials.

| Days | Focus | Required practical outcome |
|---|---|---|
| 1–3 | CLI, images, containers | Build, run, inspect, stop, remove and troubleshoot containers |
| 4–6 | Dockerfiles | Containerize static HTML, Python and Node.js applications |
| 7–9 | Layers and caching | Optimize Dockerfiles and compare build performance |
| 10–12 | Volumes and bind mounts | Demonstrate persistent data and host-to-container file sharing |
| 13–15 | Networking | Connect containers, troubleshoot DNS, and diagnose port issues |
| 16–18 | Docker Compose | Run an application and database together |
| 19–21 | Multi-stage builds | Build Java or Node.js applications using separate build and runtime stages |
| 22–23 | Distroless | Create a minimal runtime image and practise debugging without a shell |
| 24–25 | Buildx | Build multi-platform images and inspect image manifests |
| 26–27 | Security | Run as non-root, reduce capabilities, and practise secret handling |
| 28–29 | Troubleshooting | Diagnose 10 intentionally broken containers |
| 30 | Capstone | Build, test, optimize, document and publish a complete containerized application |

### Daily routine: 60–90 minutes

- **15 minutes:** Recall commands without looking at notes.
- **35 minutes:** Write or modify a Dockerfile.
- **20 minutes:** Break the application and troubleshoot it.
- **10 minutes:** Document the problem, solution and key learning.

On days when you have more time, add an extra challenge rather than repeating the same exercise.

## 16. Organize your existing Docker repository

Since your repository is intended to serve as a practical reference, I recommend this structure:

```text
Docker-Notes/
│
├── README.md
│
├── 01-Docker-Basics/
│   ├── commands.md
│   ├── image-vs-container.md
│   └── container-lifecycle.md
│
├── 02-Dockerfile/
│   ├── dockerfile-instructions.md
│   ├── python/
│   ├── nodejs/
│   └── java/
│
├── 03-Build-Optimization/
│   ├── layers-and-caching.md
│   ├── multi-stage-builds.md
│   └── distroless.md
│
├── 04-Storage/
│   ├── volumes.md
│   └── bind-mounts.md
│
├── 05-Networking/
│   ├── network-drivers.md
│   └── troubleshooting.md
│
├── 06-Docker-Compose/
│   └── application-database-stack/
│
├── 07-Buildx/
│   ├── multi-platform-builds.md
│   └── build-cache.md
│
├── 08-Security/
│   └── docker-security.md
│
├── 09-Troubleshooting/
│   ├── common-errors.md
│   └── broken-container-labs.md
│
└── 10-Interview-Preparation/
    ├── interview-questions.md
    └── practical-challenges.md
```

**My recommendation:** Keep the README short, with links to the relevant notes. Put full explanations, commands, exercises, expected outputs, and troubleshooting scenarios in the individual files.

For every practical lab, use this template:

```markdown
# Lab Name

## Objective

## Prerequisites

## Directory Structure

## Implementation

## Commands to Execute

## Expected Output

## Troubleshooting

## Challenges

## Interview Questions

## Key Takeaways
```

This makes your repository useful both for your own preparation and for someone learning Docker from scratch.

## 17. Official documentation to bookmark

Use the official documentation when practising unfamiliar commands or advanced features.

| Topic | Reference |
|---|---|
| Docker CLI | [Docker CLI reference](https://docs.docker.com/reference/cli/docker/?utm_source=chatgpt.com) |
| Dockerfiles | [Dockerfile reference](https://docs.docker.com/reference/dockerfile?utm_source=chatgpt.com) |
| Multi-stage builds | [Multi-stage builds](https://docs.docker.com/get-started/docker-concepts/building-images/multi-stage-builds/?utm_source=chatgpt.com) |
| Volumes and storage | [Docker storage](https://docs.docker.com/engine/storage/?utm_source=chatgpt.com) |
| Networking | [Docker networking](https://docs.docker.com/engine/network/?utm_source=chatgpt.com) |
| Buildx | [Buildx documentation](https://docs.docker.com/build/buildx/?utm_source=chatgpt.com) |
| Compose | [Docker Compose](https://docs.docker.com/compose/?utm_source=chatgpt.com) |
| Build secrets | [Build secrets](https://docs.docker.com/build/building/secrets/?utm_source=chatgpt.com) |
| Distroless | [Google Distroless repository](https://github.com/GoogleContainerTools/distroless?utm_source=chatgpt.com) |

## 18. Return to this ChatGPT conversation

**Bookmark this conversation for your Docker practice sessions.** You can also open [ChatGPT](https://chatgpt.com/?utm_source=chatgpt.com), but that is the general ChatGPT homepage, not a direct link to this particular thread.

For a reliable shortcut to this conversation:

1. Copy the URL from your browser's address bar while this conversation is open.
2. Save it in your browser bookmarks.
3. Alternatively, add the conversation URL to the top of your repository's `README.md`.

You can then return here whenever you need help with a Docker lab, an error message, or an interview question.

---

## Your first assignment

Don't try to master everything today. Start with these five tasks:

- [ ] Build the NGINX image from Lab 01.
- [ ] Change the HTML and rebuild the image.
- [ ] Run two containers and investigate their port mappings.
- [ ] Create a named volume, write a file, replace the container, and verify that the data persists.
- [ ] Create a user-defined bridge network and demonstrate container-to-container communication.

**The standard I want you to aim for:** Given an application and a blank directory, you should eventually be able to write its Dockerfile, build an optimized image, run it securely, persist its data, connect it to other services, and diagnose a failed deployment without copying commands from a tutorial.

That's the difference between *knowing Docker theory* and *being able to use Docker as a DevOps engineer*.
