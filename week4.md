## Question.
- run the container httpd so it is accessible on port 8080 of the host machine
- create or update an HTML file inside the container to display the  message "AMAN IS LEARNING DOCKER"
- verify the o\p using a cmd line
- properly stop and remove the container after testing ????????

# solution 

PS C:\Users\shiva> docker run -d -p 8080:80 --name myhhtp httpd
4eecf6b65ab09d3316b671a81754d45688852764131fe4c941f904fac465685a

PS C:\Users\shiva> docker exec -it 4eecf6b65ab09d3316b671a81754d45688852764131fe4c941f904fac465685a /bin/bash

root@4eecf6b65ab0:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs  modules

root@4eecf6b65ab0:/usr/local/apache2# cd htdocs
root@4eecf6b65ab0:/usr/local/apache2/htdocs# ls
index.html

root@4eecf6b65ab0:/usr/local/apache2/htdocs# echo "hello Shivam Sharma" > index.html
root@4eecf6b65ab0:/usr/local/apache2/htdocs# exit
exit
PS C:\Users\shiva> docker stop 4eecf6b65ab09d3316b671a81754d45688852764131fe4c941f904fac465685a
4eecf6b65ab09d3316b671a81754d45688852764131fe4c941f904fac465685a
PS C:\Users\shiva> docker rm 4eecf6b65ab09d3316b671a81754d45688852764131fe4c941f904fac465685a
4eecf6b65ab09d3316b671a81754d45688852764131fe4c941f904fac465685a



## DOCKER RUN OPTIONS, ENVIRONMENT VARIABLES, AND PORT MAPPING
# Docker Run Command
- The docker run command is used to create and start a container from a Docker image.

Explanation
- The docker run command is one of the most important and frequently used commands in Docker. It performs multiple operations in a single step, including pulling the image (if not available locally), creating a container, and starting it. It also allows users to configure how the container behaves using various options.

The general syntax of the command is:
- docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Here, the options define how the container should run, the image specifies the base for the container, and the optional command overrides the default command defined in the image.

## Important Docker Run Options

# Interactive Mode (-it)
- The -it option runs a container in interactive mode with a terminal.

Explanation: 
- This option combines two flags: -i (interactive) and -t (terminal). It allows the user to interact with the container through a command-line interface. This is commonly used for debugging or running commands inside the container.

Example : 
docker run -it ubuntu /bin/bash
- This command opens a terminal inside the Ubuntu container.

# Detached Mode (-d)
- The -d option runs a container in the background.

Explanation : 
- When a container is run in detached mode, it executes in the background and does not occupy the terminal. This is useful for running services such as web servers or databases.

Example: 
- docker run -d nginx
# Naming a Container (--name)
- The --name option assigns a custom name to a container.

Explanation : 
- By default, Docker assigns random names to containers. Using the --name option makes it easier to identify and manage containers.

Example: 
- docker run --name mycontainer nginx

# Automatic Removal (--rm)
- The --rm option automatically removes the container after it stops.

Explanation : 
- This is useful for temporary containers where data persistence is not required. It helps in maintaining a clean system by removing unused containers.

# Port Mapping (-p)
- The -p option maps a port on the host system to a port inside the container.

Explanation: 
- Containers run in an isolated environment, so applications inside them are not accessible from outside by default. Port mapping allows external users to access services running inside a container by linking a host port to a container port.

Syntax: 
- docker run -p <host_port>:<container_port> IMAGE

Example: 
- docker run -p 8080:80 nginx
This maps port 8080 on the host to port 80 inside the container.

Internal Flow : 
- Browser request → Host port
- Host port → Docker
- Docker → Container port
- Container → Application

# Environment Variables (-e)
- The -e option is used to set environment variables inside a container.

Explanation:
- Environment variables are key-value pairs that allow configuration of applications without modifying the image. They are widely used to pass configuration data such as database credentials, ports, and environment modes.

Example:
docker run -e MY_VAR=value nginx

Importance: 
- Avoids hardcoding configuration
- Enables dynamic behavior
- Improves flexibility
- Used in cloud and microservices environments
# Volume Mounting (-v)
- The -v option is used to mount a directory or volume from the host system into the container.

Explanation:
- This allows data to persist even after the container is removed. It also enables sharing of files between the host and the container.

Example:
- docker run -v mydata:/app/data ubuntu

# Environment Variables(Detailed)
- Environment variables are dynamic values used by applications to configure their behavior at runtime.

Explanation:
- Instead of embedding configuration directly into the application or image, environment variables allow developers to change settings without rebuilding the image. This makes applications more flexible and easier to manage.

Uses of Environment Variables: 
- Store configuration values
- Manage application settings
- Store sensitive data (like passwords)
- Control runtime behavior
Example Scenario:

A database container requires:
- Username
- Password
- Database name

These values can be passed using environment variables instead of modifying the image.
- Multiple Environment Variables

We can pass multiple variables in one command:
- docker run -e APP_ENV=production -e APP_VERSION=1.0 nginx

Using Environment File:
Instead of passing variables individually, they can be stored in a file:

Example .env file:
- DB_HOST=localhost
- DB_USER=root
- DB_PASS=secret

Run container:
- docker run --env-file .env myapp

Key Concept:
- Environment variables are created inside the container and do not affect the host system.

# Port Mapping 
- Port mapping is the process of exposing a container’s internal port to the host system.

Explanation:
- By default, containers are isolated and their services are not accessible externally. Port mapping bridges this gap by allowing external traffic to reach the application inside the container.

Without Port Mapping:
- docker run nginx

Result:
- Container runs
- Application is not accessible from browser

With Port Mapping
- docker run -p 8080:80 nginx

Result:
- Application accessible at localhost:8080

Important Points:
- Host port can be different from container port
- Required for web applications
- Essential for exposing services

=> Combined Example
Running a container with multiple options:
- docker run -dit --name myapp -p 8080:80 -e ENV=production nginx
Explanation

-d -> runs in background
-it -> allows interaction
--name -> assigns name
-p -> exposes port
-e -> sets environment variable

## CONTAINER INTERACTION AND DOCKER VOLUMES
# Container Interaction
- Container interaction refers to the process of accessing, monitoring, and managing a running container to execute commands, inspect its state, and debug applications.

Explanation:
- When a container is running, it operates in an isolated environment. However, developers and system administrators often need to interact with it to perform tasks such as debugging, checking logs, monitoring performance, or modifying files. Docker provides several commands that allow users to enter a container, execute commands inside it, and retrieve information about its state.

- Container interaction is essential for troubleshooting and maintaining applications in real-world environments. It helps in understanding how the application behaves inside the container and allows administrators to take corrective actions when needed.

# Accessing a Running Container
- docker exec : The docker exec command is used to run a command inside a running container.

Explanation:
- This command allows users to access the internal environment of a container without stopping it. It is commonly used to open a shell inside the container or execute specific commands.

Example:
- docker exec -it <container_id> bash : This command opens a terminal inside the running container, allowing direct interaction.

# Monitoring Containers
- docker logs : The docker logs command is used to view the output logs of a container.

Explanation 
- Logs provide information about the application’s execution, including errors, warnings, and normal output. This is useful for debugging and monitoring application behavior.

Example:
- docker logs <container_id>

# docker top
- The docker top command displays the running processes inside a container.

Explanation: 
This command helps in understanding which processes are active within the container and how they are behaving.

# docker stats
- The docker stats command shows real-time resource usage of containers.

Explanation:
- It provides metrics such as CPU usage, memory usage, and network I/O, which are useful for performance monitoring.

# Inspecting Containers
- docker inspect : The docker inspect command provides detailed information about a container in JSON format.

Explanation:
- This command reveals configuration details such as network settings, volume mounts, environment variables, and resource limits. It is useful for debugging and understanding container configuration.

# Importance of Container Interaction

Container interaction is important because it allows:
- Debugging of applications
- Monitoring system performance
- Access to container environment
- Verification of configurations
- Troubleshooting errors

## Docker Volumes
- A Docker volume is a storage mechanism that allows data to persist independently of containers.

Explanation:
- By default, any data created inside a container is lost when the container is removed. This is because containers are ephemeral in nature. Docker volumes solve this problem by providing persistent storage that exists outside the container’s lifecycle.

Volumes are managed by Docker and can be attached to one or more containers. They ensure that important data is not lost when containers are stopped or deleted.

# Need for Docker Volumes
Explanation:
- Containers are designed to be temporary, and their internal storage is not permanent. In real-world applications, data such as databases, logs, and user files must be preserved even after the container is removed.

Without volumes:
- Data is lost when container is deleted
- Applications cannot store persistent data

With volumes:
- Data persists across container restarts
- Multiple containers can share data
- Backup and restore become easier

# Volume Commands
- docker volume create : Creates a new Docker volume.
Example:
docker volume create myvolume

- docker volume ls : Lists all available volumes.

- docker volume inspect : Displays detailed information about a volume.

- docker volume rm : Removes a Docker volume.

# Using Volumes with Containers
- Volumes can be attached to containers using the -v option in the docker run command.

Explanation:
- When a volume is mounted, the specified directory inside the container is linked to the volume. Any data written to this directory is stored in the volume and persists even if the container is deleted.

Example: 
- docker run -d -v myvolume:/app/data ubuntu
Explanation: 
- myvolume is the Docker volume
- /app/data is the directory inside the container

Any data written to /app/data will be stored in the volume.

# Data Persistence
- Data persistence means that data remains available even after the container is stopped or removed. Volumes provide this capability by storing data outside the container.

# Bind Mounts
- A bind mount is a type of storage where a directory from the host system is directly mounted into a container.

Explanation: 
- Unlike volumes, which are managed by Docker, bind mounts use specific directories from the host system. This is commonly used in development environments where developers need to edit files on the host and see changes reflected inside the container.

Example:
- docker run -v /host/path:/container/path ubuntu
## Volume vs Bind Mount
Feature	            Volume	        Bind Mount
Managed by Docker   Yes	            No
Use case	        Production	    Development
Host dependency	    No	            Yes
Data location	    Docker-managed	Host directory