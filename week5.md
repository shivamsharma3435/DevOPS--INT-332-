# Container Lifecycle Management
Run a container (creates and starts it) 
- docker run -d -p 80:80 --name my_container nginx
Start a stopped container 
- docker start <container_id/name>
Stop a running container 
- docker stop <container_id/name>
Restart a container 
- docker restart <container_id/name>
Pause a container 
- docker pause <container_id/name>


Copying a file from container to host
step1 -> docker run -d --name task nginx
step2 -> docker exec -it task /bin/bash
step3 -> mkdir Project
step4 -> cd Project
step5 -> echo "I am Learning Docker" > newfile.txt
step6 -> cat newfile.txt
step7 -> docker attach task
step8 -> docker cp task:/Project/newfile.txt "C:\Users\shiva\Desktop\notes.txt"



## DOCKER NETWORKING, DNS, AND CONTAINER COMMUNICATION
# Introduction to Docker Networking
- Docker networking is the system that enables communication between containers, between containers and the host, and between containers across multiple hosts.

Explanation:
- Containers are isolated by default, which means they cannot communicate with each other or external systems unless explicitly configured. Docker networking provides the mechanisms to connect containers and allow data exchange. It ensures that applications running in different containers can interact securely and efficiently.

- Networking is a critical component in containerized environments, especially in microservices architectures, where multiple services need to communicate with each other to function as a complete system.

# Need for Docker Networking
Explanation:
- In modern applications, different components such as frontend, backend, and database are often deployed in separate containers. For the application to function properly, these containers must communicate with each other.

Without Docker networking:
- Containers remain isolated
- Services cannot communicate
- Application functionality breaks

With Docker networking:
- Containers can communicate using IP or names
- Services are accessible across containers
- System becomes modular and scalable

## Types of Docker Networks
- Docker provides different types of network drivers to manage communication between containers.

# Bridge Network (Default Network)
- A bridge network is the default network type in Docker that allows containers on the same host to communicate with each other.

Explanation:
- When a container is created without specifying a network, it is automatically connected to the default bridge network. Containers on the same bridge network can communicate using IP addresses. However, communication using container names is limited unless a custom bridge network is created.
- Bridge networks are suitable for standalone applications running on a single host.

Key Points: 
- Default network
- Communication via IP
- Suitable for single-host environments
- Isolation from other networks

# Custom Bridge Network
- A custom bridge network is a user-defined network that provides better communication features than the default bridge network.

Explanation:
- Custom bridge networks allow containers to communicate using container names instead of IP addresses. They also provide better isolation and automatic DNS resolution, making them more suitable for real-world applications.

Example: 
- docker network create mynetwork

Run containers:
- docker run -d --network mynetwork --name container1 nginx
- docker run -d --network mynetwork --name container2 alpine

Benefits:
- Name-based communication
- Better isolation
- Improved management

# Host Network
- The host network allows a container to share the host system’s network stack.

Explanation:
- In this mode, the container does not get its own IP address. Instead, it uses the host’s IP address and network interfaces. This removes network isolation but improves performance because there is no network translation.

Key Points:
- No separate IP
- Uses host network directly
- High performance
- Limited isolation

# Overlay Network
- An overlay network is a network that connects containers across multiple Docker hosts.

Explanation:
- Overlay networks are used in distributed systems and container orchestration platforms such as Docker Swarm. They allow containers running on different machines to communicate as if they were on the same network.

Key Points:
- Multi-host communication
- Used in clustered environments
- Enables distributed applications

## Container Communication
Explanation:
- Containers can communicate with each other in different ways depending on the network configuration. In bridge networks, containers typically communicate using IP addresses. In custom networks, they can communicate using container names.
- Communication is essential in applications where different services interact, such as a web server connecting to a database.

# DNS in Docker
- Docker provides a built-in DNS system that allows containers to resolve each other using names instead of IP addresses.

Explanation:
- In user-defined networks, Docker automatically assigns DNS entries to containers. This allows containers to communicate using their names, simplifying networking and reducing dependency on IP addresses.

Example:
If two containers are connected to the same network:

Container name: backend
Another container can access it using: backend
Importance:
- Simplifies communication
- Avoids manual IP management
- Improves flexibility

# Linking Containers (Legacy Method)
- Linking containers is an older method used to connect containers using the --link option.

Explanation:
- Before custom networks were introduced, Docker used linking to enable communication between containers. This method is now deprecated and replaced by user-defined networks.

Example:
- docker run -d --name db nginx
- docker run -it --name app --link db:database alpine
Limitation:
- Limited functionality
- Not scalable
- Deprecated in modern Docker usage

## Network Commands
# docker network ls
- Lists all available networks

# docker network create
- Creates a new network

# docker network inspect
- Displays details about a network

# docker network rm
- Removes a network

## Real-World Example
Scenario:

A web application consists of:
- Frontend container
- Backend container
- Database container

Explanation:
- All containers are connected to the same network. The frontend communicates with the backend using its container name, and the backend communicates with the database in the same way. This setup allows seamless communication between services.

## ADVANCED DOCKER CONCEPTS, IMAGE CREATION, AND REGISTRIES
# Introduction to Advanced Docker Concepts
Explanation:
- After understanding basic Docker operations such as running containers, managing volumes, and networking, the next step is to understand how Docker images are actually built, how data is stored internally, and how images are shared securely across systems. These advanced concepts include image creation using Dockerfiles, the copy-on-write mechanism, and container registries.
- These concepts are essential for real-world DevOps workflows, where applications are not just run but also built, versioned, and deployed across distributed environments.

#  Image Creation using Dockerfile
- A Dockerfile is a text file that contains a set of instructions used to automatically build a Docker image.

Explanation: 
- Instead of manually configuring a container every time, Docker allows developers to define the entire environment using a Dockerfile. This file contains step-by-step instructions such as selecting a base image, copying files, installing dependencies, and defining how the application should run.
- When the Dockerfile is executed, Docker builds an image by processing each instruction and creating layers. This ensures consistency, repeatability, and automation in application deployment.

Common Instructions in Dockerfile:
- FROM
Specifies the base image for the container.

Example:
FROM nginx:alpine

- WORKDIR
Sets the working directory inside the container.

- COPY
Copies files from the host system to the container.

- RUN
Executes commands during image build (e.g., installing packages).

- EXPOSE
Defines the port on which the application will run.

- CMD
Specifies the default command to run when the container starts.

Example Dockerfile:

FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80

Explanation:
- Uses a lightweight Nginx base image
- Copies a webpage into the container
- Exposes port 80 for web access

# Building an Image
- The process of converting a Dockerfile into an image.

Command:
- docker build -t myapp:v1 .
Explanation:
- -t assigns a name and tag
- . indicates current directory

Running the Image:
- docker run -d -p 8080:80 myapp:v1

# Copy-on-Write (CoW) Mechanism
- Copy-on-Write is a storage mechanism in Docker where changes made by a container are stored in a separate writable layer without modifying the original image layers.

Explanation:
- Docker images are made up of multiple read-only layers. When a container is created, a new writable layer is added on top of these layers. Any changes made during container execution are written only to this top layer.
- If a file from a lower layer is modified, it is first copied into the writable layer and then modified. This ensures that the original image remains unchanged.

Key Concepts:
- Base image layers are read-only
- Container layer is writable
- Changes are isolated from image
- Benefits of Copy-on-Write
- Efficient storage usage
- Faster container creation
- Image integrity is preserved
- Multiple containers can share same image

# Docker Registries
- A Docker registry is a storage and distribution system used to store and share Docker images.

Explanation:
- Registries allow users to push images from their local system and pull them on other systems. This is essential for sharing applications across teams and deploying them in different environments.

## Types of Docker Registries
# Docker Hub
- Docker Hub is the default public registry for Docker images.

Explanation:
It provides:
- Public repositories
- Official images
- Private repositories

# GitHub Container Registry (GHCR)
- A registry service provided by GitHub for storing container images.

# Private Registry
- A self-hosted or enterprise-level registry used for storing private images.

Explanation:
- Organizations use private registries to store sensitive or proprietary applications securely.

Example:
- docker run -d -p 5000:5000 --name registry registry:2

# Image Tagging and Pushing
- Tagging an Image : Tagging assigns a new name to an image for uploading to a registry.
Command:
- docker tag nginx username/myimage:latest

- Pushing an Image : Uploading an image to a registry.
Command:
- docker push username/myimage:latest

- Pulling an Image : Downloading an image from a registry.
Command: 
- docker pull nginx

# Authentication and Access Control
- To push images to registries, users must authenticate themselves. Instead of using passwords, access tokens are commonly used for security.

Example:
docker login
