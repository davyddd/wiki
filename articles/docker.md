# Docker and Compose

## Table of Contents

- [Docker](#docker)
- [Docker Compose](#docker-compose)

## Docker

**Docker** is a software platform designed for developing, delivering, and running applications. 
For virtualization, Docker uses a hypervisor based on the kernel of the host operating system (hereafter referred to as the OS).

**Note**: Do not be surprised if Docker consumes a lot of resources when virtualizing a Linux-based OS on a system with a non-Linux kernel.

Advantages of using Docker:

- _Environment reproducibility_. Ensures that the application runs on any operating system.

- _Workspace isolation_. Prevents conflicts between software versions.

- _Maintains host OS cleanliness_. Some applications require installing system packages that are later hard to remove. 
With Docker, all dependencies are installed inside an image, which can be removed with a single command.

- _Simple build and run process_. Running the application comes down to a single command 
(excluding the installation of containerization and orchestration systems).

**Dockerfile** is a text file that describes how to build an environment for an application — 
including its base system, dependencies, and configuration.

The result of building a Dockerfile is an **image** composed of layers — each corresponding to a command in the Dockerfile. 
Every layer is a read-only filesystem snapshot. Together, these layers form a unified filesystem that is used at runtime. 
Docker’s layered architecture allows multiple images on the same host to share common layers, 
avoiding duplication and saving disk space.

![Docker Image Layers](https://github.com/davyddd/wiki/blob/main/media/docker/docker-image-layers.jpg)

Figure 1 — Visual representation of layer sharing in Docker images.

A Docker **container** is a running instance of a Docker image, containing the application, its runtime, system tools, 
libraries, and configuration files. Containers are isolated from the host system and from each other, 
yet they share the host OS kernel, which makes them more efficient than virtual machines.

![Docker Image Layers](https://github.com/davyddd/wiki/blob/main/media/docker/difference-docker-image-and-container.png)

Figure 2 — Difference Between Docker Image and Container.

The **rules** of writing a Dockerfile:

- _Minimal base image_. The base image should contain only what’s necessary to run the main application. 
A smaller image means faster builds and reduced surface for vulnerabilities.

- _Exact base image version_. Never use the latest tag. It can lead to unpredictable behavior. For example, a new version of the base image might include a different set of software, which could cause your build to break — or worse, introduce bugs in a runtime.

- _Immutability of the base image_. Never run apt-get upgrade. Doing so defeats the purpose of pinning the base image tag — your build may yield different results each time.

- _One process per container_. A single container should run a single process. You do not need to include the entire application stack — such as the database, web server, and application — in one image. This is because Docker monitors the process with PID 1 inside the container. If that process dies, Docker restarts the container. If you run multiple applications in one container, or your main application is not running as PID 1, Docker might not detect failures properly.

- _Dependency installation_. Each image layer is cached. Instead of copying the entire project early in the build, copy only your dependency file (e.g., requirements.txt, project,toml, package.json). This way, the package installation step (e.g., pip install -r requirements.txt) will be cached and only re-executed when the dependency file changes.

- _Optimized layer structure_. Each instruction in a Dockerfile (RUN, COPY, ADD, etc.) creates a new image layer. 
Instead of minimizing the number of layers, it’s more important to group commands based on caching and reuse logic:

  - Group base dependencies and system-level commands (like apt update, installing utilities, etc.) into a single layer near the top of the Dockerfile.
  This layer can be cached and reused across builds, as long as the base image and dependencies remain unchanged.

  - Place application-specific commands (COPY, RUN pip install, etc.) in separate instructions near the end of the Dockerfile. 
  This ensures that when your code changes, only these layers are rebuilt — while the more stable base layers remain untouched.

  - Avoid splitting a single logical operation into multiple RUN instructions. 
  This creates unnecessary layers and increases image size. Instead, combine related commands using `&&`.

**Example** Dockerfile for a Python application:

```dockerfile
FROM python:3.12.6-slim

RUN mkdir -p /app/src && \
    apt-get update && \
    apt-get install -y git libpq-dev gcc python3-dev libffi-dev musl-dev make libevent-dev

WORKDIR /app/src

COPY requirements.txt /app/
RUN pip install -r requirements.txt

COPY . /app

CMD python manage.py runserver
```

Basic Docker **commands**:

- `docker ps` – lists all running containers (`-a` shows all containers, including stopped ones; `-q` shows only container IDs).

- `docker exec -it <container id> <command>` – runs a command inside the container.

- `docker logs <container id>` – shows application logs from the container.

- `docker kill <container id>` – stops a running container (`docker kill $(docker ps -q)` stops all running containers).

- `docker rm <container id>` – removes a container (-f forces removal).

- `docker images` – lists all images.

- `docker rmi <image id>` – removes an image (`-f` forces removal).

## Docker Compose
