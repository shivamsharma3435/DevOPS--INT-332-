## Container Image
- Definition:
A container image is a read-only, executable package that contains everything required to run an application, including the application code, runtime, libraries, dependencies, and configuration files.

- Explanation:
A container image acts as a blueprint or template from which containers are created. It defines what the application environment should look like and ensures that the application runs consistently across different systems. Since the image is read-only, it cannot be modified once it is created. Any changes result in the creation of a new image rather than altering the existing one.

This immutability is important because it guarantees consistency and reliability. Developers can build an image once and use it in multiple environments such as development, testing, and production without worrying about unexpected behavior.

Key Characteristics of Container Images:
- Read-only and immutable
- Contains application and dependencies
- Platform-independent (within compatible environments)
- Used to create containers
Example:
An image like python:3.11-slim includes:

A minimal Linux operating system
Python 3.11 runtime
Standard libraries required for Python

## Image Layers
- Definition:
Image layers are individual, stacked components that together form a complete container image. Each layer represents a change or instruction applied during the image creation process.

- Explanation:
When a container image is built, it is not created as a single file. Instead, it is composed of multiple layers stacked on top of each other. Each instruction in a Dockerfile (such as installing software or copying files) creates a new layer.

These layers are immutable and are cached by the system. This means that if a layer has not changed, it does not need to be rebuilt, which significantly improves efficiency.

Key Properties of Image Layers
- Immutable: Once created, layers cannot be modified
- Reusable: Multiple images can share common layers
- Cached: Unchanged layers are reused during builds
- Stacked structure: Layers are combined to form a complete image
Working of Layers:

When building an image:
- Base layer (operating system) is created
- Additional layers are added (libraries, dependencies)
- Final layer contains the application code

When a container runs:
- A writable layer is added on top of these read-only layers

Benefits of Image Layers
- Faster Builds
- If only one layer changes, Docker rebuilds only that layer instead of the entire image.
- Reduced Storage
- Common layers are shared across multiple images, saving disk space.
- Faster Deployment
- Only missing layers are downloaded from the registry.

## Container Image vs Container
- Explanation:
A container image is a static definition of an application, while a container is a running instance of that image.

Key Differences:
- Image	Container
- Read-only	Writable
- Static	Running
- Blueprint	Execution
- Stored in registry	Runs on host
## Image Registry
Definition:
- An image registry is a centralized system used to store, manage, version, and distribute container images.

Explanation:
- An image registry acts as a repository where container images are stored and shared. It allows developers to upload (push) images and users to download (pull) them. Registries are essential in DevOps workflows because they ensure that the same image can be used across different environments, maintaining consistency.

Importance of Image Registry

Without a registry:
- Each system must build images separately
- No consistency between environments
- No version control

With a registry:
- Build once, use everywhere
- Faster deployment
- Version management
- Easy rollback

## Types of Image Registries
# Public Registry
Definition:
- A registry where images are openly available to everyone.

Explanation:
- Public registries allow users to share and download images freely. These are commonly used for base images and open-source projects.

Example: Docker Hub

# Private Registry
Definition:
- A registry with restricted access, used within organizations.

Explanation:
- Private registries are used to store proprietary applications and ensure security. Access is controlled using authentication and authorization mechanisms.

Examples:
- AWS Elastic Container Registry (ECR)
- Azure Container Registry (ACR)
- GitHub Container Registry (GHCR)

## Image Naming Convention
Definition:
Container images follow a standard naming format:
- <registry>/<namespace>/<image>:<tag>

Explanation:
- Each part of the name provides specific information:

Registry: Location of the image
Namespace: User or organization name
Image: Application name
Tag: Version of the image

Example: docker.io/user/webapp:v1

docker.io → registry
user → namespace
webapp → image name
v1 → version

## Tags and Versioning
Definition:
- Tags are labels used to identify different versions of the same image.

Explanation:
- Tags help in managing different versions of an application. For example, an application may have multiple versions such as v1, v2, or latest. Using tags allows developers to control which version is deployed and provides the ability to roll back to a previous version if needed.

## Image Distribution Process
Explanation:
- The image distribution process describes how images move from development to production.

Steps:
- Developer builds an image locally
- Image is pushed to a registry
- Registry stores the image
- Servers pull the image
- Containers are created from the image

## Push and Pull Operations
* Push Operation : Uploading an image from local system to registry.
- Requires authentication
- Only new layers are uploaded

* Pull Operation : Downloading an image from registry.
- Uses caching
- Faster if layers already exist
## Role of Image Registry in CI/CD
- In DevOps pipelines, image registries act as an intermediary between development and deployment stages.

Workflow:
- Code is pushed to version control
- CI system builds the image
- Image is pushed to registry
- CD system pulls the image
- Application is deployed
Key Concept : 
- The registry acts as a bridge between Continuous Integration and Continuous Deployment.

## Comparison: Public vs Private Registry
Feature	  Public Registry	Private Registry
Access ->  Open	            Restricted
Usage  ->  Open-source	    Enterprise
Security-> Low               High
Cost	-> Often free	    Paid/Managed


## Introduction to Docker
- Docker is an open-source platform used to develop, package, distribute, and run applications inside containers. It simplifies application deployment by ensuring that applications run consistently across different environments.

Explanation:
- Docker provides a complete ecosystem for containerization. It allows developers to create container images, store them in registries, and run containers on any system that has Docker installed. By using Docker, developers can eliminate environment-related issues and ensure that applications behave the same way in development, testing, and production environments.

- Docker uses operating system-level virtualization and relies on underlying Linux kernel features such as namespaces and control groups to manage containers efficiently.

# Docker Architecture
- Docker architecture refers to the internal structure and components of Docker that work together to build, manage, and run containers.

Explanation :
- Docker follows a client-server architecture. The Docker client communicates with the Docker daemon, which performs the actual work of building images, running containers, and managing system resources.

Key Components of Docker Architecture:
- Docker Client : The Docker client is the command-line interface (CLI) used by users to interact with Docker.

Explanation:
Users execute commands such as docker run, docker pull, and docker build through the client. These commands are sent to the Docker daemon for execution. The client can communicate with the daemon either locally or remotely.

- Docker Daemon : The Docker daemon is a background service that manages Docker objects such as images, containers, networks, and volumes.

Explanation:
The daemon listens for API requests from the Docker client and processes them. It is responsible for building images, running containers, managing storage, and handling networking. It acts as the core engine of Docker.

- Docker Images : Docker images are read-only templates used to create containers.

Explanation:
Images contain the application code and dependencies required to run a container. They are built using Dockerfiles and stored in registries. When a container is created, it is based on an image.

- Docker Containers : A container is a running instance of a Docker image.

Explanation:
Containers are lightweight and isolated environments where applications run. Each container operates independently but shares the host system’s kernel.

- Docker Registry : A Docker registry is a repository used to store and distribute Docker images.

Explanation:
Docker Hub is the default public registry. Users can push images to a registry and pull them when needed. Registries enable sharing and versioning of images.

# Working of Docker Architecture:

The workflow of Docker architecture can be understood as follows:
- The user enters a command using the Docker client
- The client sends the request to the Docker daemon
- The daemon processes the request
- If required, it pulls the image from the registry
- The daemon creates and runs the container
This architecture ensures that Docker operations are efficient and scalable.

# Docker Lifecycle
- The Docker lifecycle describes the stages through which a container goes, from creation to execution and termination.

Stages of Docker Lifecycle
- Build Stage : In this stage, a Docker image is created using a Dockerfile. The image includes all the necessary components required to run the application.

- Ship Stage : The image is pushed to a Docker registry so that it can be accessed by other systems. This ensures portability and consistency.

- Run Stage : The image is pulled from the registry and used to create and run containers. The application becomes operational at this stage.

# Container States
A container can exist in different states during its lifecycle:
- Created: Container is initialized but not running
- Running: Container is actively executing
- Paused: Execution is temporarily stopped
- Stopped: Container has been terminated

## Basic Docker Commands
# Image Management Commands

- docker pull : Downloads an image from a registry.
Explanation:
This command retrieves an image from Docker Hub or another registry and stores it locally.

- docker images : Lists all available images on the system.

- docker rmi : Removes a Docker image.

# Container Management Commands
- docker run : Creates and starts a container from an image.

Explanation:
This is one of the most important Docker commands. It combines multiple actions such as creating, starting, and executing a container.

- docker stop : Stops a running container.

- docker start : Starts a stopped container.

- docker rm : Removes a container.

# Example Commands

Running a container:
- docker run ubuntu

Running with terminal:
- docker run -it ubuntu /bin/bash

Stopping container:
- docker stop <container_id>

Removing container:
- docker rm <container_id>

Removing image:
- docker rmi ubuntu

# Additional Useful Commands
- docker ps : Lists running containers
- docker ps -a : Lists all containers (running and stopped)
- docker system prune : Removes unused data such as stopped containers and unused images

## Real-World Example
To run a web server using Docker:

Pull image:
- docker pull nginx
Run container:
- docker run -d -p 8080:80 nginx
Access in browser:
- http://localhost:8080