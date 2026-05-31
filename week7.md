### UNIT III: MICROSERVICES WITH DOCKER COMPOSE
### WEEK 7: MICROSERVICES ARCHITECTURE AND NEED

## INTRODUCTION TO ARCHITECTURAL STYLES

Software architecture has evolved significantly over the years. Understanding the shift from monolithic applications to microservices is crucial for modern DevOps practices.
In a traditional setup, applications were built as a single, cohesive unit. Today, the demand for high availability, rapid deployment, and massive scalability has driven the adoption of microservices.

## MONOLITHIC ARCHITECTURE
A monolithic architecture is a unified model for designing a software program.
Characteristics of Monolithic Apps:
- Single Codebase: All features, UI, business logic, and database access layers are combined.
- Tightly Coupled: Components are heavily dependent on each other.
- Shared Memory: Modules share the same memory space and processing resources.

Challenges of Monolithic Architecture:
- Scaling is Difficult: To scale one component (e.g., payment processing), you must scale the entire application, which is highly resource-inefficient.
- Slow Deployments: Any small change requires recompiling and deploying the entire application.
- Blast Radius: A memory leak in one module can crash the entire system.
- Technology Lock-in: It is extremely difficult to adopt new languages or frameworks.

## MICROSERVICES ARCHITECTURE
Microservices architecture breaks down a large application into a suite of modular services. Each service is organized around a specific business capability.
Key Features:
- Loosely Coupled: Services can be updated or replaced independently.
- Independently Deployable: You can deploy the 'User Auth' service without touching the 'Order Processing' service.
- Polyglot Programming: Different services can be written in different programming languages (e.g., Node.js for API gateways, Python for Data Science modules).

## ADVANTAGES OF MICROSERVICES
- Scalability: You can scale only the services that require more resources. This horizontal scaling saves massive amounts of cloud compute costs.
- Isolation: Process isolation means a crash in one service does not bring down the entire system.
- Agility: Smaller, cross-functional teams can own a specific microservice from development to deployment (DevOps culture).
- Faster Time-to-Market: CI/CD pipelines run much faster on smaller codebases.

## API GATEWAY
An API Gateway is a server that acts as an API front-end, receiving API requests, enforcing throttling and security policies, passing requests to the back-end service, and then passing the response back to the requester.
Why do we need an API Gateway?
- It provides a single entry point for clients.
- It handles cross-cutting concerns like SSL termination, authentication, and logging.
- It reduces the number of requests clients must make by aggregating data from multiple microservices.

============
PRACTICALS 
============

### Practical 1: Designing an Nginx API Gateway
**Question/Scenario:**
You are migrating from a monolith to microservices. You have an 'inventory' service running on port 8081 and a 'shipping' service running on port 8082. Write an Nginx configuration that acts as an API gateway, routing `/api/inventory` traffic to the inventory service and `/api/shipping` to the shipping service.

**Solution / Code:**
```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    upstream inventory_service {
        server localhost:8081;
    }

    upstream shipping_service {
        server localhost:8082;
    }

    server {
        listen 80;
        server_name myapi.com;

        location /api/inventory/ {
            proxy_pass http://inventory_service/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api/shipping/ {
            proxy_pass http://shipping_service/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```
*Explanation:* This configuration uses the `upstream` directive to define backend microservices. The `location` blocks map URI paths to the corresponding microservices.

### Practical 2: Containerizing a Simple Microservice
**Question/Scenario:**
Write a minimal Node.js Express microservice that returns "Inventory Service Running" on the root path. Then, write a Dockerfile to package this microservice using the Alpine Linux base image to keep the image size small.

**Solution / Code:**

**`app.js`**
```javascript
const express = require('express');
const app = express();
const port = 8081;

app.get('/', (req, res) => {
    res.json({ service: 'Inventory', status: 'Running', version: '1.0' });
});

app.listen(port, () => {
    console.log(`Inventory service listening at http://localhost:${port}`);
});
```

**`Dockerfile`**
```dockerfile
FROM node:18-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 8081
CMD ["node", "app.js"]
```
*Explanation:* We use `node:18-alpine` for a minimal footprint. The `EXPOSE` directive documents the port, and `CMD` provides the default execution command for the container.
---

## WINDOWS PRACTICAL SETUP

**Prerequisites:** Docker Desktop running, WSL2 enabled.

```powershell
docker info
docker compose version
```

### Windows Lab — Run inventory microservice
```powershell
cd labs\unit-2-dockerfile
docker build -t inventory-svc:1.0 .
docker run -d --name inventory -p 8081:3000 inventory-svc:1.0
curl.exe http://localhost:8081
```