### UNIT VI: CI/CD WITH JENKINS
### WEEK 16: JENKINS PIPELINES & JENKINSFILE

## 1. DECLARATIVE VS SCRIPTED PIPELINE SYNTAX
A Pipeline in Jenkins is a suite of plugins that supports implementing continuous delivery pipelines.
There are two Groovy-based syntaxes:
- Declarative Pipeline: The modern approach. It provides a strict, structured, and simple syntax using blocks like `pipeline`, `stages`, `steps`, and `post`. It is easier to read and validate.
- Scripted Pipeline: The legacy approach. It is basically raw Groovy code running inside Jenkins. It offers immense flexibility and complex control flow (if/else loops) but can become unmaintainable spaghetti code.

## 2. JENKINSFILE STRUCTURE (Declarative)
A standard Jenkinsfile resides in the root of the Git repository.
Key blocks:
- `pipeline`: The root element.
- `agent`: Defines WHERE the pipeline executes (e.g., `agent any`, `agent { label 'linux' }`).
- `stages`: Contains a sequence of one or more `stage` directives.
- `stage`: Represents a logical phase of the pipeline (e.g., "Build", "Test").
- `steps`: The actual tasks executed within a stage (e.g., running shell commands via `sh`).
- `post`: Actions that run at the very end of the pipeline depending on the status (e.g., `success`, `failure`, `always`).

## 3. PARAMETERS AND ENVIRONMENT VARIABLES
Pipelines can be parameterized to accept dynamic inputs when a user clicks "Build with Parameters" in the UI.
Environment variables can be defined globally in the `environment` block and accessed via \`${env.VAR_NAME}\`.

## 4. JENKINS MULTI-BRANCH PIPELINES
A Multi-branch Pipeline project type automatically scans a Git repository. For every branch or Pull Request it finds that contains a `Jenkinsfile`, it automatically creates a corresponding Jenkins job. This is essential for modern GitFlow practices.

## 5. MANAGING ARTIFACTS
After compiling code, the resulting binaries (JAR files, ZIPs, or test reports) are ephemeral and will be deleted when the workspace is cleaned. The `archiveArtifacts` step saves these files permanently on the Jenkins master so users can download them from the build dashboard.

============
PRACTICALS 
============

### Practical 3: Writing a Declarative Pipeline
**Question/Scenario:**
Write a `Jenkinsfile` using Declarative syntax. It should run on any agent. It must have two stages: "Build" (which runs `make build`) and "Test" (which runs `make test`). Finally, regardless of success or failure, it must print "Pipeline Complete" in a post block.

**Solution / Code:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Starting compilation...'
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                echo 'Executing unit tests...'
                sh 'make test'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline Complete!'
        }
    }
}
```

### Practical 4: Archiving Build Artifacts
**Question/Scenario:**
In your Jenkins pipeline, the "Build" stage produces a file named `app-release.zip` inside the `target/` directory. Add a step to securely archive this file so it appears on the Jenkins build results page.

**Solution / Code:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build and Archive') {
            steps {
                sh 'mkdir -p target && touch target/app-release.zip'
                
                // Archive the generated artifact
                archiveArtifacts artifacts: 'target/*.zip', fingerprint: true
            }
        }
    }
}
```
*Explanation:* The `archiveArtifacts` step copies files matching the glob pattern from the ephemeral agent workspace to the persistent Jenkins controller storage. `fingerprint: true` allows tracking the artifact across different jobs.
### Windows Lab — Jenkinsfile
Create Jenkinsfile with Checkout, Build, Test, Package stages.

=======================================
### UNIT VI: CI/CD WITH JENKINS
(DOCKER, MAVEN & JENKINS INTEGRATION)
=======================================

## 1. DOCKER INSIDE JENKINS AGENTS
A major headache in CI/CD is "agent dependency management." If Project A needs Node 14 and Project B needs Node 18, installing both on the same Jenkins agent causes conflicts.
Solution: Use Docker as the execution environment.
In a Declarative Pipeline, you can set the agent to a Docker image. Jenkins will dynamically spin up the container, mount the Git workspace inside it, execute your `sh` steps inside the container, and destroy it when finished. This guarantees a clean, reproducible build environment.

## 2. JENKINS AND MAVEN INTEGRATION
Maven can be integrated tightly into Jenkins.
- Global Tool Configuration: Jenkins admins can configure specific Maven and JDK versions.
- Running Maven: You execute Maven using `sh 'mvn clean package'`.
- Test Reports: The `junit` step in Jenkins parses `target/surefire-reports/*.xml` files generated by Maven and creates beautiful visual trend graphs on the Jenkins dashboard showing passed/failed tests over time.

## 3. CI/CD DEPLOYMENT FLOWS & TRIGGERS
How do builds start?
- Polling (pollSCM): Jenkins asks Git every minute "are there changes?". This wastes resources.
- Webhooks (Push): GitHub sends an HTTP POST payload to Jenkins the instant a developer pushes code. This is instant and efficient.
Deployments are often executed by running shell scripts that invoke `kubectl apply` for Kubernetes, or Ansible playbooks for server configuration.

## 4. PIPELINE SHARED LIBRARIES
In a company with 50 microservices, writing 50 identical Jenkinsfiles violates the DRY (Don't Repeat Yourself) principle.
Jenkins Shared Libraries allow you to write reusable Groovy functions (e.g., `standardJavaBuild()`), store them in a separate Git repository, and import them globally. A project's Jenkinsfile then becomes a single line calling the library function.

## 5. BACKUP & PIPELINE BEST PRACTICES
- Jenkins Configuration as Code (JCasC): Configure Jenkins system settings using YAML instead of clicking UI buttons.
- Backup: The `$JENKINS_HOME` directory holds everything. Back it up regularly.
- Keep Pipelines Dumb: Don't put 500 lines of complex shell scripting inside a Jenkinsfile. Write a `deploy.sh` script in the repo, and have Jenkins simply run `sh './deploy.sh'`. This allows developers to test the exact deployment script locally.

============
PRACTICALS 
============

### Practical 5: Executing Pipelines Inside Docker Containers
**Question/Scenario:**
Write a Jenkinsfile where the build environment dynamically uses the `maven:3.8.4-openjdk-11` Docker container to compile the code, ensuring that Maven does not need to be installed on the host Jenkins agent machine.

**Solution / Code:**
```groovy
pipeline {
    agent {
        docker { 
            image 'maven:3.8.4-openjdk-11' 
            args '-v $HOME/.m2:/root/.m2' // Cache maven dependencies on host
        }
    }
    stages {
        stage('Compile') {
            steps {
                sh 'mvn --version'
                sh 'mvn clean package -DskipTests'
            }
        }
    }
}
```
*Explanation:* Jenkins automatically handles pulling the image and wrapping the `steps` inside a `docker run` execution.

### Practical 6: Parsing JUnit Test Reports
**Question/Scenario:**
After running `mvn test`, Maven generates XML test reports. Add a `post` block to your pipeline that uses the Jenkins JUnit plugin to parse these XML files so the results appear in the Jenkins UI.

**Solution / Code:**
```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                // Ensure tests run but don't immediately fail the pipeline if a test fails
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'mvn test'
                }
            }
        }
    }
    post {
        always {
            // Parse the generated XML reports
            junit 'target/surefire-reports/*.xml'
        }
    }
}
```
*Explanation:* The `junit` step reads the XML files and populates the "Test Result" visualization in the Jenkins dashboard. `catchError` ensures the pipeline proceeds to the `post` block even if a test fails.
### Windows Lab — Jenkins + Maven + Docker pipeline
Configure Maven/Docker in Jenkins Global Tool Configuration.