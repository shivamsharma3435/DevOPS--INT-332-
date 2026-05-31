### UNIT V: CONTINUOUS INTEGRATION WITH GITHUB ACTIONS
### WEEK 12: RUNNERS, DOCKER & CLOUD DEPLOYMENTS

## 1. RUNNERS: GITHUB-HOSTED VS SELF-HOSTED
- GitHub-Hosted Runners: VMs fully managed by GitHub. They are spun up fresh for your job and destroyed immediately after. They come pre-installed with Docker, Node, Python, etc. They are extremely convenient but have minute quotas on private repositories.
- Self-Hosted Runners: You install the GitHub Actions Agent software on your own infrastructure (AWS EC2, Raspberry Pi, On-Prem servers). You get full control over hardware and no execution time limits, but you are responsible for OS updates, security, and cleaning up workspaces between runs.

## 2. RUNNER SECURITY & MANAGEMENT
When using Self-Hosted runners, NEVER allow them on public repositories where anyone can open a Pull Request. A malicious PR could execute arbitrary code on your internal corporate network.

## 3. DOCKER & GITHUB ACTIONS
Building and publishing Docker containers is the most common use case for modern CI/CD.
GitHub hosted runners (`ubuntu-latest`) natively support Docker commands out of the box.

## 4. PUSHING TO DOCKER HUB
You can use official marketplace actions (`docker/build-push-action`) to build images securely. Authentication secrets (like passwords and tokens) should NEVER be hardcoded. They must be stored in "GitHub Repository Secrets" and accessed via `${{ secrets.SECRET_NAME }}`.

## 5. GITHUB CONTAINER REGISTRY (GHCR)
GitHub provides its own container registry. It is deeply integrated with GitHub Actions. The implicit `${{ secrets.GITHUB_TOKEN }}` provided to every workflow run can be used to automatically authenticate and push images to GHCR without needing to set up external Docker Hub credentials.

## 6. DEPLOYING TO SERVERS/CLOUD
Once artifacts/images are built, Actions can trigger deployments.
- SSH Actions: To SSH into a remote Linux server and run `git pull && docker-compose up`.
- Cloud CLI: Actions come pre-installed with AWS CLI, Azure CLI, and Google Cloud SDK to trigger serverless deployments or Kubernetes rolling updates.

============
PRACTICALS 
============

### Practical 5: Building and Pushing to Docker Hub
**Question/Scenario:**
Write a workflow job that checks out the code, logs into Docker Hub using secrets `DOCKER_USER` and `DOCKER_PASS`, builds the Dockerfile in the repository, and pushes it to `myuser/myapp:latest`.

**Solution / Code:**
```yaml
jobs:
  docker-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USER }}
          password: ${{ secrets.DOCKER_PASS }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v3
        with:
          context: .
          push: true
          tags: myuser/myapp:latest
```

### Practical 6: Deploying via SSH
**Question/Scenario:**
After your image is pushed, you need to tell your production server to pull the latest image and restart. Write a step using the `appleboy/ssh-action` to connect to your server and execute a restart script.

**Solution / Code:**
```yaml
steps:
  - name: Execute deployment script over SSH
    uses: appleboy/ssh-action@master
    with:
      host: ${{ secrets.SERVER_HOST }}
      username: ${{ secrets.SERVER_USER }}
      key: ${{ secrets.SERVER_SSH_KEY }}
      script: |
        cd /opt/myapp
        docker pull myuser/myapp:latest
        docker-compose down
        docker-compose up -d
```
*Explanation:* This connects to the remote server securely using an SSH private key stored in GitHub Secrets, and executes raw shell commands on the remote machine to restart the Docker stack.
### Windows Lab — Docker in CI
Add a `docker build` step tagged with `${{ github.sha }}` in your workflow.


=====================================
### UNIT VI: CI/CD WITH JENKINS
(JENKINS FOUNDATIONS & ARCHITECTURE)
=====================================

## 1. INTRODUCTION TO JENKINS
Jenkins is the most widely used open-source automation server. It provides hundreds of plugins to support building, deploying, and automating any project. Unlike GitHub Actions which is a SaaS, Jenkins is self-managed infrastructure.

## 2. JENKINS ARCHITECTURE (MASTER/AGENT MODEL)
Jenkins operates on a distributed architecture to handle heavy workloads:
- Master (Controller) Node: The central control unit. It serves the web UI, handles HTTP requests, parses configurations, schedules jobs, and monitors agents. It should NOT execute heavy builds itself to prevent crashing.
- Agent (Worker/Slave) Nodes: Machines connected to the master that actually execute the jobs. Agents can be static VMs, bare-metal servers, or dynamically provisioned Docker/Kubernetes pods.

## 3. INSTALLATION & UI OVERVIEW
Jenkins is a Java application distributed as a `.war` file. It can run standalone via `java -jar jenkins.war`, be deployed on Tomcat, or run inside a Docker container.
The Classic UI is function-focused, while the "Blue Ocean" plugin UI provides modern, graphical visualizations of CI pipelines.

## 4. PLUGINS MANAGEMENT
Jenkins' true power is its plugin ecosystem. Out of the box, Jenkins does very little.
Plugins add integrations for Git, Maven, Docker, AWS, Slack notifications, and Pipeline syntax.
Plugins are managed via the web UI (`Manage Jenkins -> Manage Plugins`) or via CLI tools during automated provisioning.

## 5. SECURITY, USERS, ROLES
Security is critical because Jenkins executes arbitrary code and holds deployment secrets.
- Authentication: Can be internal, or integrated with LDAP/Active Directory, GitHub OAuth, or SAML.
- Authorization (RBAC): Using the Role-Based Strategy plugin, administrators can define who can read jobs, trigger builds, or access the underlying system configuration.

## 6. FREESTYLE VS PIPELINE JOBS
- Freestyle Jobs: Configured by clicking through web UI forms. Easy to start, but impossible to track changes (no version control for the job config).
- Pipeline Jobs: Configured by writing code (a `Jenkinsfile`) stored alongside the application source code in Git (Pipeline-as-Code). This is the industry standard.

============
PRACTICALS 
============

### Practical 1: Installing Jenkins via Docker
**Question/Scenario:**
You need to spin up a local Jenkins controller for testing without installing Java on your laptop. Write the docker command to run the official `jenkins/jenkins:lts` image, exposing the UI on port 8080 and persisting the data.

**Solution / Code:**
```bash
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins-master \
  jenkins/jenkins:lts
```
*Explanation:* Port 8080 serves the Web UI. Port 50000 is used by agents to communicate with the master via JNLP/TCP. The named volume `jenkins_home` ensures job configurations aren't lost when the container stops.

### Practical 2: Extracting the Initial Admin Password
**Question/Scenario:**
When you first boot Jenkins, it locks the UI and asks for an "initial admin password" generated during installation. Assuming Jenkins is running in a Docker container named `jenkins-master`, how do you retrieve this password via the CLI?

**Solution / Code:**
```bash
docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
```
*Explanation:* The `docker exec` command allows you to run a command inside a running container. We use `cat` to read the auto-generated security token file required to unlock the Jenkins setup wizard.
### Windows Lab — Jenkins in Docker
```powershell
docker volume create jenkins_home
docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --name jenkins-lab jenkins/jenkins:lts
docker exec jenkins-lab cat /var/jenkins_home/secrets/initialAdminPassword
```

