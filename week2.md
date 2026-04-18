WEEK 2: CONTAINERIZATION AND CONTAINER RUNTIME
## INTRODUCTION TO CONTAINERIZATION
- Containerization is a lightweight virtualization technique in which applications and their dependencies are packaged together into a single unit called a container, which can run consistently across different environments.

Key Concept:
Unlike virtual machines, containers:
- Do not include a full operating system
- Share the host operating system kernel
- Contain only required libraries and dependencies

Technical Explanation
A container includes:
- Application code
- Runtime environment
- Libraries and dependencies
- Configuration files
However, it does not include a separate OS, which makes it lightweight.

Real-World Analogy

Container = Packed lunch
It contains only what is needed, not the entire kitchen.

## NEED FOR CONTAINERIZATION
Containerization emerged due to limitations of virtualization.

Problems with Virtualization:
- Each VM requires a full OS
- High memory and CPU usage
- Slow startup time
- Difficult scalability
- Portability issues

How Containerization Solves These Problems
- Lightweight
- Containers use fewer resources because they share the host OS.
- Fast Startup
- Containers start in seconds, unlike VMs.
- High Scalability
- You can run many containers on a single system.
- Portability

Containers run the same across development, testing, and production.

Evolution of Application Deployment
Monolithic Applications
Single large application
Deployed on physical servers
Virtualized Systems
Applications split into components
Hosted on VMs
Microservices Architecture
Applications divided into small services
Each service runs in a container

## CHARACTERISTICS OF CONTAINERS
1. Lightweight
- No separate OS, minimal overhead

2. Fast Boot Time
- Starts almost instantly

3. Portable
- Runs anywhere with same behavior

4. Scalable
- Easily create or destroy containers

5. Efficient Resource Usage
- Multiple containers share system resources

# CONTAINER VS VIRTUAL MACHINE
Feature	        Virtual Machine	   Container
OS	            Separate OS	       Shared OS
Size	        Large	           Small
Startup	        Slow	           Fast
Resource usage	High	           Low
Portability	    Limited	           High

# CONTAINER RUNTIME
- A container runtime is the software responsible for creating, running, and managing containers.

Key Role : 
- It converts a container image into a running container.

Simple Explanation
- Image = blueprint
- Runtime = engine that runs it

Responsibilities of Container Runtime
- Create isolated environment
- Start application process
- Apply resource limits
- Monitor container lifecycle
- Stop or delete containers

# NEED FOR CONTAINER RUNTIME

A container image is:
- Read-only
- Static

To execute it, a runtime is required to:
- Allocate system resources
- Create isolation
- Execute application inside container

Without runtime, containers cannot run.

# TYPES OF CONTAINER RUNTIMES
1. High-Level Runtimes
- User-facing tools that manage container lifecycle and provide additional features.

Examples: 
- Docker Engine
- containerd
- CRI-O

Functions: 
- Pull images from registry
- Manage networking
- Manage storage
- Start/stop containers

2. Low-Level Runtimes
- Responsible for directly interacting with the operating system to run containers.

Example: 
- runc
Functions: 
- Interface with Linux kernel
- Handle namespaces
- Apply cgroups
- Execute container process

Key Difference: 
High-Level Runtime	 Low-Level Runtime
User-facing	         System-level
Manages lifecycle	 Executes container
Example: Docker	     Example: runc

# HOW CONTAINERS WORK (INTERNAL FLOW)
- Developer creates container image
- Runtime loads the image
- Runtime creates isolated environment
- Application starts running inside container

# ROLE OF CONTAINERS IN DEVOPS
- Containers are a core part of modern DevOps because they enable:

1. Continuous Integration
- Same environment for building and testing

2. Continuous Deployment
- Deploy identical containers in production

3. Microservices Architecture
- Each service runs independently

4. Automation
- Easy integration with CI/CD pipelines

# REAL-WORLD SCENARIO

Suppose you build an application on your laptop:
- Without containers → may fail on server
- With containers → runs exactly the same everywhere

# IMPORTANT EXAM CONCEPTS
Difference Between Image and Container: 

Image:	          Container:
Static	          Running instance
Read-only	      Writable
Blueprint	      Execution

# Process Isolation
- Process isolation is a mechanism that ensures each container runs independently as if it is the only process on the system, even though multiple containers share the same operating system.

Explanation: 
- In a normal operating system, all processes can potentially interact with each other, which may lead to conflicts or security issues. In containerized environments, multiple applications run on the same host system. Therefore, it is essential to isolate processes so that one container cannot interfere with another.

- Process isolation creates the illusion that each container has its own dedicated system. Even though containers share the same OS kernel, they behave as independent environments. This isolation is achieved using Linux kernel features called namespaces, which restrict what a container can see and access.

Importance of Process Isolation: 
- Prevents interference between containers
- Improves security
- Ensures stability of applications
- Allows multiple applications to run safely on the same system

# Namespaces
- Namespaces are Linux kernel features that provide isolation by partitioning system resources so that each container only sees its own resources.

Explanation: 
- Namespaces work by creating separate views of system resources for different processes. Each container is assigned its own set of namespaces, which restrict its visibility to only its own processes, files, and network interfaces. As a result, a container behaves as if it is running on a separate machine, even though it is sharing the host system.

- In simple terms, namespaces create isolated environments within a single operating system.

Key Concept: 
- “Namespaces create isolation by hiding system resources from other containers.”

## Types of Namespaces
# PID Namespace (Process Isolation)
- The PID namespace isolates process IDs so that each container has its own process tree.

Explanation:
- Inside a container, processes are assigned their own IDs starting from 1. These processes are not visible outside the container, and the container cannot see processes running on the host or in other containers.

Example:
A web server container may have:

PID 1 → main application
Worker processes → handle requests
Logger process → manage logs

These processes are only visible within the container.

# Network Namespace
- The network namespace provides each container with its own network stack.

Explanation:
Each container gets its own:
- IP address
- Network interfaces
- Port space

This allows multiple containers to run applications on the same port without conflict.

Example:
Multiple containers can run web servers on port 80:
- Container 1 → 172.x.x.x:80
- Container 2 → 172.x.x.x:80

# Mount Namespace
- The mount namespace isolates the file system of each container.

Explanation:
Each container has its own view of the file system. Changes made inside a container do not affect the host system or other containers.

# UTS Namespace
- The UTS namespace allows each container to have its own hostname and domain name.

Explanation:
This helps in identifying containers independently, especially in distributed systems.

# User Namespace
- The user namespace maps container users to different users on the host system.

Explanation:
It enhances security by preventing containers from running as root users on the host system, even if they appear as root inside the container.

# Summary of Namespaces
Namespace	 Purpose
PID	         Process isolation
Network	     Separate networking
Mount	     File system isolation
UTS	         Hostname isolation
User	     Security and user mapping

# Control Groups (cgroups)
- Control Groups (cgroups) are Linux kernel features used to limit, control, and monitor the resource usage of processes or containers.

Explanation:
- While namespaces provide isolation, they do not control how much resource a container can use. Without proper control, one container could consume excessive CPU or memory, affecting other containers and the host system.

- cgroups solve this problem by enforcing limits on resource usage. They ensure fair distribution of system resources and prevent any single container from overwhelming the system.

Key Concept:
- “Namespaces isolate resources, cgroups control resource usage.”

# Need for cgroups
In a multi-container environment:
- Multiple applications share the same system
- Some applications may consume more resources

Without cgroups:
- One container could use all CPU or memory
- Other containers may crash
- System performance degrades
With cgroups:
- Resource limits are enforced
- System remains stable
- Fair resource sharing is ensured

# Resources Controlled by cgroups
1. CPU
- Limits how much CPU a container can use and assigns CPU shares.

2. Memory
- Sets maximum memory usage and can terminate containers if limits are exceeded.

3. Disk I/O
- Controls read and write speeds to prevent excessive disk usage.

4. Network (Indirect Control)
- Managed through traffic shaping mechanisms.

# Working of cgroups
- cgroups organize processes into groups and apply limits to each group. When a container is created, it is assigned to a specific cgroup, which defines how much CPU, memory, and other resources it can use.

- The system continuously monitors resource usage and ensures that containers do not exceed their allocated limits.

## Namespaces vs cgroups
Feature	    Namespaces	    cgroups
Purpose	    Isolation	    Resource control
Function	Hide resources	Limit usage
Focus	    Visibility	    Allocation