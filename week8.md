### UNIT III: MICROSERVICES WITH DOCKER COMPOSE
### WEEK 8: DOCKER COMPOSE YAML STRUCTURE & ENVIRONMENT

## 1. INTRODUCTION TO DOCKER COMPOSE
While Docker CLI allows you to run single containers, real-world microservices require running multiple containers simultaneously (e.g., a web server, an application server, and a database).
Docker Compose is an orchestration tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services.

## 2. YAML STRUCTURE
YAML (YAML Ain't Markup Language) is a human-readable data serialization language.
Key rules:
- It uses indentation (spaces, NEVER tabs) to define structure and hierarchy.
- Key-value pairs are separated by colons.
- Lists/Arrays are denoted by a dash (-).
- It is highly sensitive to syntax formatting.

## 3. WRITING docker-compose.yml
The `docker-compose.yml` file typically consists of four main top-level keys:
- `version`: Defines the Compose file format version (e.g., '3.8'). Determines which features are supported.
- `services`: The core section where you define the individual containers (microservices) that make up your app.
- `networks`: Defines custom virtual networks to allow secure communication between specific containers.
- `volumes`: Defines persistent storage mechanisms so data is not lost when containers restart.

## 4. BUILD VS IMAGE FIELDS
Within a service definition, you must specify how the container is created:
- `image`: Used when you want to pull a pre-built image from a registry like Docker Hub (e.g., `image: postgres:13`).
- `build`: Used when you want Docker Compose to build the image locally from a Dockerfile. You specify the `context` (the directory containing the Dockerfile).

## 5. ENVIRONMENT VARIABLES
Environment variables are critical in the Twelve-Factor App methodology. They allow you to change configuration (like database passwords or API endpoints) without altering the container image.
In Docker Compose, you can define them directly under the `environment` key or pass an `.env` file using the `env_file` directive.

## 6. SECRETS AND CONFIGS
For sensitive data like production database passwords or TLS certificates, standard environment variables are not secure enough. Docker Compose supports `secrets`, which securely injects sensitive files into the container at runtime.

============
PRACTICALS
============

### Practical 3: Writing a Multi-Service docker-compose.yml
**Question/Scenario:**
Create a `docker-compose.yml` file that defines two services: a Redis cache using the pre-built `redis:alpine` image, and a web application built from a local directory `./webapp`. The web application should map host port 5000 to container port 5000.

**Solution / Code:**
```yaml
version: '3.8'

services:
  cache:
    image: redis:alpine
    restart: always

  webapp:
    build: 
      context: ./webapp
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    depends_on:
      - cache
```
*Explanation:* This configuration ensures the cache pulls a lightweight image, while the webapp is built from local source code. `depends_on` ensures the cache starts before the web application.

### Practical 4: Environment Variables and .env Files
**Question/Scenario:**
You need to pass a database user and password to a PostgreSQL container securely. Write a `.env` file containing the credentials, and write the `docker-compose.yml` snippet that loads these variables dynamically.

**Solution / Code:**

**`.env`**
```env
DB_USER=admin_user
DB_PASS=SuperSecretPassword123
DB_NAME=production_db
```

**`docker-compose.yml`**
```yaml
version: '3.8'

services:
  database:
    image: postgres:14-alpine
    env_file:
      - .env
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASS}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```
*Explanation:* The `env_file` directive loads the `.env` file, and the `${VAR}` syntax interpolates those values into the container's environment natively.
---

## WINDOWS PRACTICAL SETUP

**Prerequisites:** Docker Desktop running, WSL2 enabled.

```powershell
docker info
docker compose version
```

### Windows Lab — Multi-service Compose
```powershell
cd labs\unit-3-wordpress
docker compose config
docker compose up -d
```