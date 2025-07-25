# Docker and Compose

## Table of Contents

- [Docker](#docker)
- [Docker Compose](#docker-compose)
- [Installation guide](#installation-guide)

## Docker

**Docker** is a software platform designed for developing, delivering, and running applications. 
For virtualization, Docker uses a hypervisor based on the kernel of the host operating system (hereafter referred to as the OS).

**Note**: Do not be surprised if Docker consumes a lot of resources when virtualizing a Linux-based OS on a system with a non-Linux kernel.

Advantages of using Docker:

- _Environment reproducibility_. Ensures that the application runs on any operating system.

- _Workspace isolation_. Prevents conflicts between software versions.

- _Keeps the host OS clean_. Some applications require installing system packages that are later hard to remove. 
With Docker, all dependencies are installed inside an image, which can be removed with a single command.

- _Simple build and run process_. Running the application comes down to a single command 
(excluding the installation of containerization and orchestration systems).

**Dockerfile** is a text file that describes how to build an environment for an application — 
including its base system, dependencies, and configuration.

The result of building a Dockerfile is an **image** composed of layers — each corresponding to a command in the Dockerfile. 
Every layer is a read-only filesystem snapshot. Together, these layers form a unified filesystem that is used at runtime. 
Docker’s layered architecture allows multiple images on the same host to share common layers, 
avoiding duplication and saving disk space.

<img src="https://raw.githubusercontent.com/davyddd/wiki/refs/heads/main/media/docker-and-compose/docker-image-layers.jpg" width="650" title="Docker image layers">

Figure 1 — Visual representation of layer sharing in Docker images.

A Docker **container** is a running instance of a Docker image, containing the application, its runtime, system tools, 
libraries, and configuration files. Containers are isolated from the host system and from each other, 
yet they share the host OS kernel, which makes them more efficient than virtual machines.

<img src="https://raw.githubusercontent.com/davyddd/wiki/refs/heads/main/media/docker-and-compose/difference-docker-image-and-container.png" width="500" title="Difference Docker image and container">

Figure 2 — Difference between Docker image and container.

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

**Docker Compose** is a tool for defining and running multi-container Docker applications (an orchestration system). 
It uses a configuration file to manage orchestration settings, describing how each container should be launched 
and how they interact with each other.

Basic **tags* used in service configuration:

- `image` – the name of a prebuilt image from a registry (such as Docker Hub).

- `build` – the relative path to the directory containing the Dockerfile. 
For a single service, you can specify either image or build, but not both.

- `command` – the command that will be used to start the container. This overrides the CMD instruction from the Dockerfile.

- `volumes` – mounts host directories into the container’s filesystem. Multiple containers can share access to the same directory. 
This tag is used mainly in local development to enable hot reload. It is generally avoided in production environments, 
as file access in mounted volumes is slower and may affect web application performance.

- `ports` – maps host ports to container ports. The ports don’t have to match; for example, 
you can expose port 80 in the container as port 8080 on the host.

- `depends_on` – lists dependent containers. The service will wait for these containers to start before launching.

- `environment` – defines environment variables to be set inside the container.

**Example** `docker-compose.yml` for a Python application:

```yaml
x-server-template: &server-template
    build:
        context: ./
        dockerfile: ./Dockerfile
        args:
            PYTHONBREAKPOINT: import ipdb;ipdb.set_trace
    volumes:
        - .:/app
    depends_on:
        - redis
        - postgres
    environment:
        REDIS_URL=redis://redis/1?socket_connect_timeout=2
        POSTGRES_URL=postgresql+psycopg://postgres_user:postgres_password@postgres:5432/postgres_db

services:
    server:
        <<: *server-template
        command: python manage.py runserver
        ports:
            - 8000:8000

    worker:
        <<: *server-template
        command: python manage.py runworker --processes 2 --threads 20

    redis:
        image: redis/redis-stack:7.2.0-v13

    postgres:
        image: postgres:16.2-alpine
        environment:
            POSTGRES_DB: postgres_db
            POSTGRES_USER: postgres_user
            POSTGRES_PASSWORD: postgres_password
```

**Note**: Compose automatically creates a dedicated virtual network and runs a built-in DNS server that maps each service name 
to its IP address within that network. This means you don’t need to know the exact IP address assigned to a service — 
you can simply refer to it by the name defined in the Compose file. For example, a database service named `postgres` 
will be accessible to other containers at the address `postgres`.

Basic Compose **commands**:

- `docker-compose build <service name>` – builds the image for a specific service. If no service is specified, all services are built.

- `docker-compose up` – starts all services/containers. To build and start at once, use the `--build` flag.

- `docker-compose run <service name>` – runs a specific service and allows you to override the container’s default startup command. 
For example, `docker-compose run --rm --service-ports server bash`, where the `--rm` flag is used to remove the container after it exits, 
helping keep the host system clean; the `--service-ports` flag is required to forward ports to the host, 
since run does not do this by default — even if the `ports` section is defined in the config file.

**Notes**:

- By default, Compose looks for a configuration file named `docker-compose.yml` in the root directory.

- To use a configuration file with a custom name, use the `-f` flag. 
For example, `docker-compose -f docker-compose.develop.yml up`

- To override specific settings without modifying the main file, you can use a `docker-compose.override.yml` file.
This file is typically added to `.gitignore` and often contains personal credentials to private services (e.g., Postgres).

## Installation guide

### MacOS

On macOS, Docker and Compose are bundled together as part of [Docker Desktop](https://docs.docker.com/desktop/setup/install/mac-install/).

[OrbStack](https://orbstack.dev/) is available as an alternative to Docker Desktop. According to its developers, 
it features a more optimized virtual machine and improved volume performance.

### Linux

On Linux, Docker and Compose need to be installed separately.
Follow the official installation instructions for your distribution:
- [Docker installation guide](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Docker-compose installation guide](https://docs.docker.com/compose/install/)

Running `docker` without `sudo`

```bash
sudo groupadd docker
sudo gpasswd -a ${USER} docker
newgrp docker
sudo service docker restart
```

Verify that `docker` works without `sudo`:
```bash
docker run hello-world
```

Running `docker-compose` without `sudo`

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
