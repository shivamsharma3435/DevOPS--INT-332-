### GitHub Container Registry (GHCR) 
# Overview
- GitHub Container Registry (GHCR) allows developers to store, manage, and distribute Docker container images securely using GitHub.

Key Objectives:
- Build a Docker image
- Push the image to GHCR
- Pull the image from GHCR
- Understand authentication using Personal Access Token (PAT)

# Authentication using GitHub PAT
- To interact with GHCR, authentication is required using a Personal Access Token (PAT).

Steps to Create PAT:
- Step 1: Navigate to Token Settings
Go to:
GitHub → Settings → Developer Settings → Personal Access Tokens
Direct Link: https://github.com/settings/tokens

- Step 2: Generate Token
Choose: Classic Token
Enable permissions:
write:packages
read:packages
delete:packages (optional)

- Step 3: Copy Token
Copy the token immediately
It will not be visible again

Important:
PAT acts as a password for Docker, NOT your GitHub account password.

# Creating a Demo Project
Create Project Directory:
- mkdir ghcr-demo
- cd ghcr-demo
- Create Dockerfile

Create a file named Dockerfile and add:
- FROM nginx:alpine
- COPY index.html /usr/share/nginx/html/index.html
- Create HTML File

Create index.html:
<h1>GHCR Demo Success</h1>

# Build Docker Image
Run the following command:

- docker build -t ghcr.io/yourusername/ghcr-demo .

Replace yourusername with your GitHub username

Example:
- docker build -t ghcr.io/harpreetkaur/ghcr-demo .

# Login to GHCR
Authentication is required before pushing the image.

docker login ghcr.io

Enter:
- Username: GitHub username
- Password: Personal Access Token (PAT)

Do NOT use your GitHub password.

# Push Image to GHCR
- docker push ghcr.io/yourusername/ghcr-demo
What Happens:
Local image → Uploaded → Stored in GitHub Container Registry

# Verify Image on GitHub
Go to:
GitHub → Profile → Packages

You will see:
- Image name
- Visibility (public/private)
- Version details

# Pull Image from GHCR
Remove Local Image
- docker rmi ghcr.io/yourusername/ghcr-demo
Pull Image Again
- docker pull ghcr.io/yourusername/ghcr-demo

# Run the Container
- docker run -p 8080:80 ghcr.io/yourusername/ghcr-demo
Access in Browser
http://localhost:8080

If successful, you will see:
“GHCR Demo Success”

# Key Takeaways
- GHCR integrates seamlessly with GitHub for container management
- PAT is essential for authentication
- Docker commands follow standard workflow:
- Build → Login → Push → Pull → Run
- Useful for CI/CD pipelines and deployment workflows