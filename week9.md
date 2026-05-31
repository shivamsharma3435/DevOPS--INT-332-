### UNIT III: MICROSERVICES WITH DOCKER COMPOSE
### WEEK 9: SERVICE DEPENDENCY & USE CASE DEPLOYMENTS

## 1. SERVICE DEPENDENCY ORDERING
In a microservices architecture, the order in which services start is critical. An API service will crash if it tries to connect to a database that hasn't booted up yet.
- `depends_on`: Controls startup and shutdown order in Compose.
- Note: Standard `depends_on` only waits for the container to be running, not for the application inside to be "ready". 
- Healthchecks: For true readiness waiting, you combine `depends_on` with a `condition: service_healthy`, requiring the database container to define a `healthcheck` block (like executing a `pg_isready` ping).

## 2. MULTI-CONTAINER APPS: DATABASE + BACKEND + FRONTEND
A classic 3-tier architecture involves:
- Frontend Container: Serves static assets (React/Angular) via Nginx.
- Backend Container: The API logic (Node.js/Spring Boot/Django).
- Database Container: Persistent storage (MySQL/MongoDB/Postgres).
Docker Compose places all these on a default custom bridge network, allowing them to communicate via service names (e.g., the backend can connect to `http://database:3306`).

## 3. USE CASE: WORDPRESS + MYSQL
WordPress requires a relational database to function.
- We define a `mysql` service with persistent volumes to ensure blog posts aren't lost when the container stops.
- We define a `wordpress` service that depends on the MySQL service and uses environment variables to configure the database connection automatically.

## 4. USE CASE: NODE.JS + MONGODB
The MERN/MEAN stack relies heavily on NoSQL.
- The Node.js application connects to MongoDB using a connection string like `mongodb://mongo:27017/mydb`.
- Volumes are mapped to `/data/db` inside the Mongo container to persist JSON documents.

## 5. USE CASE: JAVA SPRING BOOT + POSTGRESQL
Enterprise deployments often use Java.
- Spring Boot uses an `application.properties` file. We can override these properties using Docker environment variables (e.g., `SPRING_DATASOURCE_URL`).

============
PRACTICALS 
============

### Practical 5: WordPress and MySQL Deployment
**Question/Scenario:**
Write a full Docker Compose configuration to deploy a WordPress blog backed by a MySQL 5.7 database. Ensure that the database data is persisted using a named volume, and that WordPress uses the correct database credentials.

**Solution / Code:**
```yaml
version: '3.8'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wp_password

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_password
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data:
```

### Practical 6: Healthchecks for Dependency Ordering
**Question/Scenario:**
Your backend Node.js application crashes immediately because the PostgreSQL database takes 10 seconds to fully initialize. Write a compose snippet using a `healthcheck` on the Postgres container, and configure the backend to wait for that healthcheck before starting.

**Solution / Code:**
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend
    depends_on:
      postgres:
        condition: service_healthy
```
*Explanation:* The `pg_isready` command physically pings the database. The `backend` service will not attempt to boot until the healthcheck returns a success status.
### Windows Lab — WordPress + MySQL
```powershell
cd labs\unit-3-wordpress
docker compose up -d
docker compose ps
# Open http://localhost:8000
```

![docker compose up on Windows](assets/screenshots/unit-3/01-compose-up.svg)
*docker compose up on Windows — captured on Windows PowerShell + Docker Desktop*

![Compose services running](assets/screenshots/unit-3/02-compose-ps.svg)
*Compose services running — captured on Windows PowerShell + Docker Desktop*

![Database container logs](assets/screenshots/unit-3/03-compose-logs.svg)
*Database container logs — captured on Windows PowerShell + Docker Desktop*




=======================================
### UNIT IV: MAVEN BUILD AUTOMATION
=======================================

## 1. WHY BUILD TOOLS EXIST
In the early days of Java, compiling a project meant manually executing `javac` commands for every single source file, remembering complex classpath strings, and manually downloading JAR files from the internet.
Build tools like Apache Maven, Gradle, and Ant were created to automate this entire workflow. They provide a standardized way to build, package, and deploy applications.

## 2. PROBLEMS SOLVED BY AUTOMATED BUILDS
- "JAR Hell": Manually managing libraries leads to version conflicts. Build tools automatically resolve and download dependencies.
- Reproducibility: A project builds the exact same way on a developer's laptop as it does on the CI server.
- Standardization: Maven enforces a standard directory structure. Any Java developer can immediately understand a Maven project layout.
- Lifecycle Execution: Running a single command (`mvn clean install`) handles cleaning, compiling, testing, and packaging automatically.

## 3. DIRECTORY STRUCTURE
Maven enforces Convention over Configuration:
- `src/main/java`: The core application source code.
- `src/main/resources`: Configuration files (e.g., properties, XML, YAML).
- `src/test/java`: Unit and integration test code.
- `target/`: The ephemeral output directory where compiled `.class` files and the final `.jar`/`.war` files are stored.

## 4. PROJECT OBJECT MODEL (POM)
The `pom.xml` is the brain of a Maven project. It is an XML file that contains information about the project and configuration details used by Maven to build the project.
Key elements in a POM:
- `groupId`: Uniquely identifies your project across all projects (usually a reverse domain name, e.g., `com.mycompany`).
- `artifactId`: The name of the jar without version.
- `version`: The current version (e.g., `1.0.0-SNAPSHOT`).
- `dependencies`: The external libraries required by the project.
- `build`: Configuration for plugins and goals.

## 5. BUILD LIFECYCLE PHASES
Maven is based on the central concept of a build lifecycle. The default lifecycle consists of several phases executed sequentially:
- `validate`: Validates that the project is correct and all necessary information is available.
- `compile`: Compiles the source code of the project.
- `test`: Tests the compiled source code using a testing framework.
- `package`: Takes the compiled code and packages it in its distributable format (JAR).
- `verify`: Runs any checks on results of integration tests.
- `install`: Installs the package into the local repository (`~/.m2`).
- `deploy`: Copies the final package to a remote repository for sharing.

============
PRACTICALS 
============

### Practical 1: Generating a Maven Project from Archetype
**Question/Scenario:**
Using the Maven CLI, generate a standard Java project skeleton using the `maven-archetype-quickstart`. Set the `groupId` to `com.devops.demo` and `artifactId` to `calculator-app`.

**Solution / Code:**
```bash
mvn archetype:generate \
  -DgroupId=com.devops.demo \
  -DartifactId=calculator-app \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false
```
*Explanation:* The Archetype plugin generates the standard Maven directory structure (`src/main/java`, `pom.xml`, etc.) automatically, saving manual folder creation time.

### Practical 2: Executing the Build Lifecycle
**Question/Scenario:**
You have made changes to your Maven project. You need to delete the old compiled files, compile the new code, run the unit tests, and package the application into a JAR file. What single command achieves this?

**Solution / Code:**
```bash
mvn clean package
```
*Explanation:* The `clean` goal wipes the `target/` directory. The `package` phase sequentially triggers `validate`, `compile`, and `test` before finally zipping the compiled `.class` files into a JAR inside the `target/` directory.
---

## WINDOWS PRACTICAL SETUP

**Prerequisites:** Docker Desktop running, WSL2 enabled.

```powershell
docker info
docker compose version
```

### Windows Lab — Maven project
```powershell
cd labs\unit-4-maven
mvn -B clean verify
java -jar target\calculator-app-1.0.0-SNAPSHOT.jar
```

![Maven version on Windows](assets/screenshots/unit-4/01-mvn-version.svg)
*Maven version on Windows — captured on Windows PowerShell + Docker Desktop*

![Maven clean verify build](assets/screenshots/unit-4/02-mvn-clean-verify.svg)
*Maven clean verify build — captured on Windows PowerShell + Docker Desktop*