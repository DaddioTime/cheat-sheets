![docker](assets/hero-docker.jpg)

- [Docker Guide for Developers](#docker-guide-for-developers)
  - [1. Docker Installation](#1-docker-installation)
    - [Option 1: Install from Ubuntu Repository](#option-1-install-from-ubuntu-repository)
    - [Option 2: Install Using the Convenience Script](#option-2-install-using-the-convenience-script)
    - [Option 3: Install from Docker’s Official Repository](#option-3-install-from-dockers-official-repository)
  - [2. Verify Docker Installation](#2-verify-docker-installation)
  - [3. Basic Docker Commands](#3-basic-docker-commands)
    - [Pulling and Running Images](#pulling-and-running-images)
  - [4. Docker Networking](#4-docker-networking)
    - [Creating and Managing Networks](#creating-and-managing-networks)
  - [5. Creating a Custom Docker Image](#5-creating-a-custom-docker-image)
    - [Example Dockerfile](#example-dockerfile)
    - [start.sh Script](#startsh-script)
    - [Building and Running the Image](#building-and-running-the-image)
  - [6. Container Management Commands](#6-container-management-commands)
    - [Common Container Operations](#common-container-operations)
  - [7. Image Management Commands](#7-image-management-commands)
    - [Managing Docker Images](#managing-docker-images)
  - [8. Volumes \& Networks Management](#8-volumes--networks-management)
    - [Working with Volumes](#working-with-volumes)
    - [Working with Networks](#working-with-networks)
  - [9. Debugging \& Inspection](#9-debugging--inspection)
    - [Useful Commands for Troubleshooting](#useful-commands-for-troubleshooting)
  - [10. Bulk Container Management](#10-bulk-container-management)
    - [Managing Multiple Containers](#managing-multiple-containers)
  - [11. Executing Commands Within Containers](#11-executing-commands-within-containers)
    - [Managing Container Processes and Files](#managing-container-processes-and-files)
  - [12. Docker Image Operations](#12-docker-image-operations)
    - [Additional Commands for Images](#additional-commands-for-images)


# Docker Guide for Developers

This comprehensive guide provides step-by-step instructions for installing Docker, verifying the installation, and managing containers, images, networks, volumes, and more. Follow each section to set up and work with Docker effectively.

---

## 1. Docker Installation

### Option 1: Install from Ubuntu Repository
This method installs Docker from the default Ubuntu repositories.
```bash
sudo apt-get install docker.io -y
```

### Option 2: Install Using the Convenience Script
Download and execute Docker's convenience script.
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```
_Source: [Docker Docs: Install using the convenience script](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)_

### Option 3: Install from Docker’s Official Repository
Follow these steps to install Docker from the official repository:

1. **Set up Docker’s apt repository:**
   ```bash
   # Update package list and install prerequisites
   sudo apt-get update
   sudo apt-get install -y ca-certificates curl

   # Create a directory for Docker's keyrings
   sudo install -m 0755 -d /etc/apt/keyrings

   # Add Docker's official GPG key
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc

   # Add the Docker repository to Apt sources
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
     $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

   # Update the package index
   sudo apt-get update
   ```

  _Source: [Docker Docs: Install using the apt repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)_


2. **Install Docker Packages:**
   ```bash
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

---

## 2. Verify Docker Installation

Ensure Docker is running properly by checking its status and running a test container.

- **Check Docker service status:**
  ```bash
  sudo systemctl status docker
  ```

- **Run the hello-world container:**
  ```bash
  sudo docker run hello-world
  ```

---

## 3. Basic Docker Commands

### Pulling and Running Images

- **Pull an image from Docker Hub:**
  ```bash
  docker pull alpine
  ```

- **Run a container (detached mode):**
  ```bash
  docker run -d -t --name palermo alpine
  ```

- **Run a container with port mapping:**
  ```bash
  docker run -d -t -p 80:80 --name palermo alpine
  ```

- **List running containers:**
  ```bash
  docker ps
  ```

- **Execute a command in a running container:**
  ```bash
  docker exec -it one sh
  ```
  *Note: Use `bash` instead of `sh` if supported by the container.*

---

## 4. Docker Networking

### Creating and Managing Networks

- **Create a new network:**
  ```bash
  sudo docker network create net1
  ```

- **Run a container on the new network:**
  ```bash
  sudo docker run --rm --network net1 --name tokio busybox
  sudo docker run itd --rm --network net1 --name tokio busybox 🔴
  ```

- **Run a container with host networking:**
  ```bash
  sudo docker run -itd --rm --network host --name helsinki alpine
  sudo docker run -itd --rm -- network host --name helsinki 🔴

  ```

- **Create a macvlan network (for direct access via router):**
  ```bash
  sudo docker network create -d macvlan \
    --subnet 192.168.24.0/24 \
    --gateway 192.168.24.1 \
    -o parent=enp0s3 homenetwork

    🔴 sudo docker network creat -d macvlan --subnetz 192.168.24.0/24 --gateway 192.168.24.1 -o partner=enp0s3 homenetwork 🔴
  ```
  > *Note: Replace `enp0s3` with your actual network interface. You may need to manually assign the host IP for this setup.*

- **List all networks:**
  ```bash
  docker network ls
  ```

---

## 5. Creating a Custom Docker Image

### Example Dockerfile

Create a file named `Dockerfile` with the following content:
```dockerfile
FROM ubuntu:latest

# Set non-interactive frontend for Docker builds
ENV DEBIAN_FRONTEND=noninteractive

# Update system and install curl
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    ENV Port=8080 \
    rm -rf /var/lib/apt/lists/*

# Copy scripts or files into the image
COPY ./start.sh /start.sh
RUN chmod +x /start.sh

# Define the default command when the container starts
CMD ["/start.sh"]
```

### start.sh Script

Create a file named `start.sh` with the following content:
```bash
#!/bin/bash
echo "Starting Container..."
curl https://example.org
```

### Building and Running the Image

- **Build the image:**
  ```bash
  docker build -t one .
  ```

- **List local images:**
  ```bash
  docker images
  ```

- **Run the created image:**
  ```bash
  docker run one
  ```

---

## 6. Container Management Commands

### Common Container Operations

| Command                                | Description                                                  |
|----------------------------------------|--------------------------------------------------------------|
| `docker run <image>`                   | Create and start a new container from an image               |
| `docker run -d <image>`                | Start a container in detached (background) mode              |
| `docker run -it <image>`               | Start a container interactively (e.g., to access a shell)      |
| `docker exec -it <container> sh`       | Execute a command in a running container                     |
| `docker ps`                            | List running containers                                      |
| `docker ps -a`                         | List all containers (including stopped ones)                 |
| `docker stop <container>`              | Stop a running container                                     |
| `docker start <container>`             | Start a stopped container                                    |
| `docker restart <container>`           | Restart a container                                          |
| `docker rm <container>`                | Remove a container                                           |
| `docker logs <container>`              | Display logs from a container                                |

---

## 7. Image Management Commands

### Managing Docker Images

| Command                           | Description                                                  |
|-----------------------------------|--------------------------------------------------------------|
| `docker pull <image>`             | Download an image from Docker Hub                            |
| `docker images`                   | List local images                                            |
| `docker rmi <image>`              | Remove a Docker image                                          |
| `docker build -t <name> .`        | Build an image from a Dockerfile                             |

---

## 8. Volumes & Networks Management

### Working with Volumes

| Command                           | Description                                                  |
|-----------------------------------|--------------------------------------------------------------|
| `docker volume ls`                | List all volumes                                             |
| `docker volume create <volume>`   | Create a new volume                                          |
| `docker volume rm <volume>`       | Remove a volume                                              |
| `docker volume prune`             | Remove all dangling volumes (not used by any container)      |

### Working with Networks

| Command                           | Description                                                  |
|-----------------------------------|--------------------------------------------------------------|
| `docker network ls`               | List all networks                                            |
| `docker network create <name>`    | Create a new network                                         |
| `docker network rm <name>`        | Remove a network                                             |

---

## 9. Debugging & Inspection

### Useful Commands for Troubleshooting

| Command                                           | Description                                                 |
|---------------------------------------------------|-------------------------------------------------------------|
| `docker inspect <container/image>`                | Display detailed information (JSON format)                   |
| `docker stats`                                    | Show live resource usage (CPU, memory, etc.)                |
| `docker top <container>`                          | List processes running inside a container                    |
| `docker diff <container>`                         | Show changes made to a container’s filesystem                 |
| `docker logs -f <container>`                      | Follow real-time logs of a container                        |

---

## 10. Bulk Container Management

### Managing Multiple Containers

| Command                                                   | Description                                                  |
|-----------------------------------------------------------|--------------------------------------------------------------|
| `docker stop $(docker ps -q)`                             | Stop all running containers                                  |
| `docker rm $(docker ps -a -q)`                              | Remove all containers (both running and stopped)             |
| `docker system prune`                                     | Remove all unused data (dangling images, containers, volumes)  |
| `docker rmi -f $(docker images -q)`                        | Force-remove all images                                      |

---

## 11. Executing Commands Within Containers

### Managing Container Processes and Files

| Command                                                        | Description                                                      |
|----------------------------------------------------------------|------------------------------------------------------------------|
| `docker attach <container>`                                    | Attach to a running container                                    |
| `docker exec -it <container> /bin/bash` or `sh`                | Run an interactive shell inside a container                      |
| `docker cp <container>:<container-path> <host-path>`             | Copy files from a container to the host                          |
| `docker cp <host-path> <container>:<container-path>`             | Copy files from the host into a container                        |
| `docker export <container>`                                    | Export the contents of a container as a tar archive              |
| `docker wait <container>`                                      | Wait until the container terminates and return its exit code     |

---

## 12. Docker Image Operations

### Additional Commands for Images

| Command                                      | Description                                                  |
|----------------------------------------------|--------------------------------------------------------------|
| `docker history <image>`                     | Show the history of an image                                  |
| `docker tag <image> <tag>`                   | Tag an image                                                 |
| `docker commit <container> <image>`          | Create a new image from a container                          |
| `docker import <url>`                        | Create an image from a tarball                               |
| `docker login`                               | Log in to a Docker registry                                  |
| `docker logout`                              | Log out from a Docker registry                               |
| `docker save <image>`                        | Save an image to a tar archive                               |
| `docker load`                                | Load an image from a tar archive                             |