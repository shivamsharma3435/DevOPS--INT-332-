### UNIT IV: MAVEN BUILD AUTOMATION
### WEEK 10: DEPENDENCY MANAGEMENT AND MAVEN PLUGINS

### 1. PARENT POM
In large organizations, you often have dozens of microservices. Instead of duplicating configurations, you can create a "Parent POM". Child modules inherit dependency versions, properties, and plugin configurations from this parent, enforcing strict standardization across the entire codebase.

### 2. DEPENDENCY SCOPE
Dependencies are not always needed at all times. Maven uses "Scopes" to optimize the build:
- `compile` (Default): Required for compiling and running the app. Included in the final JAR.
- `provided`: Required for compiling, but assumed to be provided by the server at runtime (e.g., Servlet API on Tomcat).
- `runtime`: Not needed for compiling, but required for execution (e.g., MySQL JDBC Driver).
- `test`: Only needed during the test phase (e.g., JUnit, Mockito).

### 3. TRANSITIVE DEPENDENCIES & VERSION CONFLICTS
When you declare a dependency on Library A, and Library A requires Library B, Maven automatically downloads Library B. This is a transitive dependency.
Version Conflicts occur when your project brings in multiple different versions of Library B through different transitive paths. Maven resolves this using "Nearest Definition" (the version closest to your project root wins). You can also use `<exclusions>` to manually remove conflicting sub-dependencies.

### 4. USING DEPENDENCY MANAGEMENT
The `<dependencyManagement>` tag is usually placed in a Parent POM. It does NOT add dependencies to the project; it merely declares the *versions* of dependencies. When child modules declare the dependency, they omit the version tag, automatically inheriting the version from the Parent POM.

### 5. MAVEN PLUGINS
Maven is essentially a plugin execution framework. Every phase in the lifecycle is executed by a plugin.
- Compiler Plugin: Compiles Java source files. You can configure it to target specific Java versions (e.g., Java 17).
- Surefire Plugin: The testing workhorse. It executes JUnit/TestNG tests during the `test` phase and generates reports.
- Shade Plugin: By default, a Maven JAR only contains your code, not your dependencies. The Shade plugin creates an "Uber JAR" (or Fat JAR) which unpacks all dependencies and bundles them into a single, executable JAR file.

### 6. MAVEN WRAPPER (mvnw)
The Maven Wrapper is a script (`mvnw`) bundled with the project repository. It automatically downloads the correct version of Maven defined by the project. This ensures that CI/CD pipelines and new developers don't have to manually install Maven on their operating systems.

============
PRACTICALS
============

### Practical 3: Analyzing and Resolving Transitive Dependencies
**Question/Scenario:**
Your build is failing due to a conflicting version of the `guava` library. You need to visualize the dependency tree to find out which library is bringing in the wrong version of Guava. What command do you run?

**Solution / Code:**
```bash
mvn dependency:tree -Dincludes=com.google.guava:guava
```
*Explanation:* The `dependency:tree` plugin visualizes the hierarchy. The `-Dincludes` flag filters the massive output to only show paths leading to the Guava library, making debugging trivial.

### Practical 4: Creating an Executable Uber JAR with Shade
**Question/Scenario:**
Your command-line application requires the Apache Commons library. Write the plugin configuration in the `pom.xml` to use the `maven-shade-plugin` so that `mvn package` produces a single standalone executable JAR file with the main class `com.demo.Main`.

**Solution / Code:**
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <version>3.2.4</version>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>shade</goal>
          </goals>
          <configuration>
            <transformers>
              <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>com.demo.Main</mainClass>
              </transformer>
            </transformers>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```
![Maven dependency tree](assets/screenshots/unit-4/03-mvn-dependency-tree.svg)
*Maven dependency tree — captured on Windows PowerShell + Docker Desktop*

```powershell
cd labs\unit-4-maven
mvn dependency:tree
```


=================================
UNIT IV: MAVEN BUILD AUTOMATION
=================================

### 1. DOCKERIZING MAVEN-BASED APPLICATIONS
In modern DevOps workflows, the end artifact of a Java build is no longer just a JAR file; it is a Docker image containing the JRE and the JAR file.
The traditional process involves:
- Running `mvn clean package` to generate the JAR in the `target/` directory.
- Writing a `Dockerfile` in the root directory.
- Running `docker build -t myapp:1.0 .` to package the JAR into a Linux container.

### 2. WRITING THE DOCKERFILE FOR JAVA
A standard Java Dockerfile should:
- Use a lightweight JRE base image (like Alpine or Eclipse Temurin) rather than a full JDK to reduce image size.
- Define a working directory.
- Copy the generated JAR file into the image.
- Expose the application port.
- Define the `ENTRYPOINT` to execute the JAR.

### 3. MAVEN AND DOCKER PLUGINS
Instead of writing shell scripts to run Docker commands, you can integrate Docker directly into the Maven lifecycle using plugins.
- `dockerfile-maven-plugin` (Spotify - Deprecated but historically important): Builds Docker images using a local Docker daemon during the `package` phase.
- `fabric8-maven-plugin`: Extremely powerful plugin for building Docker images and generating Kubernetes manifests.
- `jib-maven-plugin` (Google): A revolutionary plugin that builds optimized Docker and OCI images for Java applications WITHOUT needing a Docker daemon (no `Dockerfile` required). It layers the image efficiently (separating dependencies from classes) for extremely fast rebuilds.

### 4. PUSHING ARTIFACTS TO REGISTRIES
Once the Docker image is built, the final step in the CI pipeline is pushing it to a Container Registry (Docker Hub, AWS ECR, GitHub Container Registry).
Plugins like Jib can be configured with registry credentials in the `pom.xml` or via environment variables to automatically push the image during the `mvn deploy` phase.

============
PRACTICALS
============

### Practical 5: Standard Dockerfile for a Maven Project
**Question/Scenario:**
You have successfully run `mvn package` and produced `target/myapp-1.0.jar`. Write a `Dockerfile` using the `eclipse-temurin:17-jre-alpine` base image to containerize this application. It runs on port 8080.

**Solution / Code:**
```dockerfile
# Use a lightweight JRE image
FROM eclipse-temurin:17-jre-alpine

# Set the working directory inside the container
WORKDIR /opt/app

# Copy the JAR from the target folder to the container
COPY target/myapp-1.0.jar app.jar

# Expose the application port
EXPOSE 8080

# Execute the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```
*To build:* `docker build -t my-java-app:1.0 .`

### Practical 6: Daemonless Docker Builds with Google Jib
**Question/Scenario:**
You are running a CI pipeline on a runner that does NOT have Docker installed. How can you use the Maven CLI and the Google Jib plugin to build a container image and push it directly to Docker Hub repository `myrepo/myapp`?

**Solution / Code:**
```bash
mvn compile jib:build \
  -Dimage=docker.io/myrepo/myapp:latest \
  -Djib.to.auth.username=myUser \
  -Djib.to.auth.password=mySecretPassword
```
*Explanation:* The `jib:build` goal connects directly to the container registry API. It compiles the Java code, separates dependencies into distinct container layers for optimization, and pushes the image over HTTPS without ever invoking a local Docker daemon.
### Windows Lab — Dockerize Maven app
```powershell
cd labs\unit-4-maven
mvn -B clean package
# Add Dockerfile with JRE base + COPY target/*.jar
```

