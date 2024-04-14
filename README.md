# Docker-Concepts-
Docker Concepts 
Link https://github.com/iam-veeramalla/Docker-Zero-to-Hero

another notes

https://github.com/mindsparkist/docker-notes

## Docker Shim

**Imagine Docker containers as isolated apartments in a large building (Docker engine).** Each apartment has its own plumbing, electricity, and (potentially) internet connection (resources). Tenants (processes) live in these apartments and can't directly interact with the building's infrastructure.

**Docker Shim acts like a concierge for these apartments.** It receives instructions from the building manager (Docker daemon) about what tenants (processes) to move in (start), evict (stop), or how much power and water (resources) to allocate to them.

Here's a breakdown of Docker Shim's role:

1. **Process Spawning:** When you run a `docker run` command, Docker daemon tells Docker Shim to create a process inside the container. Docker Shim handles the low-level details of starting that process, similar to how a concierge hands over the keys to a new tenant.

2. **Resource Allocation:** Docker daemon might instruct Docker Shim to limit the CPU, memory, or disk space available to a container process. Docker Shim enforces these restrictions, ensuring no single tenant uses more than their fair share of utilities.

3. **Isolation:** Docker Shim ensures container processes are isolated from the host system and other containers. They can't directly access the building's mainframe (host kernel) or tamper with each other's plumbing (resources).

4. **Communication Channel:** Docker Shim acts as a go-between for container processes and the Docker daemon. Processes can send signals (like ringing the concierge bell) to request actions (more power, internet access). Docker Shim relays these messages to the Docker daemon, which decides how to respond.

**In essence, Docker Shim is the bridge between the high-level Docker daemon and the low-level processes running inside containers. It ensures they can start, stop, and function correctly within their isolated environments.**

**Additional Notes:**

- Docker Shim is typically a small binary that doesn't require deep understanding for most software engineers. But it's a good idea to have a high-level concept of its role in the Docker ecosystem.
- You might not directly interact with Docker Shim in your day-to-day coding, but it plays a crucial part in making Docker containers work seamlessly.
