# Kubernetes Hello World on Mac - Step by Step Guide

## Prerequisites Installation

### Step 1: Install Docker Desktop
```bash
# Download from https://www.docker.com/products/docker-desktop/
# Or install via Homebrew:
brew install --cask docker
```
**Why:** Docker Desktop provides both Docker engine and Kubernetes cluster for local development.

### Step 2: Enable Kubernetes in Docker Desktop
- Open Docker Desktop → Settings → Kubernetes → Check "Enable Kubernetes"
- This gives you a local single-node Kubernetes cluster without additional setup.

### Step 3: Install kubectl (if not included)
```bash
brew install kubectl
```
**Why:** kubectl is the command-line tool to interact with your Kubernetes cluster.

### Step 4: Verify Installation
```bash
docker --version
kubectl version --client
kubectl cluster-info
```
**Why:** Confirms Docker and Kubernetes are properly installed and running.

## Create Your Hello World Service

### Step 5: Create Application Directory
```bash
mkdir hello-k8s && cd hello-k8s
```
**Why:** Organize all your files in one place for easy management.

### Step 6: Create a Simple Node.js App
```javascript
// app.js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Kubernetes!',
    hostname: require('os').hostname(),
    timestamp: new Date().toISOString()
  });
});

app.listen(port, () => {
  console.log(`Hello World app listening on port ${port}`);
});
```
**Why:** Simple web service that shows which pod is responding (useful for load balancing demo).

### Step 7: Create package.json
```json
{
  "name": "hello-k8s",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.0"
  },
  "scripts": {
    "start": "node app.js"
  }
}
```
**Why:** Defines Node.js dependencies and startup command for containerization.

### Step 8: Create Dockerfile
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```
**Why:** Instructions to package your app into a Docker container image.

## Build and Test Docker Image

### Step 9: Build Docker Image
```bash
docker build -t hello-k8s:v1 .
```
**Why:** Creates a container image with your app that can run anywhere Docker is installed.

### Step 10: Test Docker Container Locally
```bash
docker run -p 3000:3000 hello-k8s:v1
# Visit http://localhost:3000 in browser
# Press Ctrl+C to stop
```
**Why:** Verify your containerized app works before deploying to Kubernetes.

## Deploy to Kubernetes

### Step 11: Create Kubernetes Deployment
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-k8s-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-k8s
  template:
    metadata:
      labels:
        app: hello-k8s
    spec:
      containers:
      - name: hello-k8s
        image: hello-k8s:v1
        ports:
        - containerPort: 3000
```
**Why:** Tells Kubernetes to run 3 copies of your app for high availability.

### Step 12: Create Kubernetes Service
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-k8s-service
spec:
  selector:
    app: hello-k8s
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```
**Why:** Exposes your app to external traffic and load balances between the 3 pods.

### Step 13: Deploy to Kubernetes
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```
**Why:** Creates the deployment and service in your Kubernetes cluster.

## Test Your Microservice

### Step 14: Check Pod Status
```bash
kubectl get pods
kubectl get services
```
**Why:** Verify all 3 pods are running and service is created successfully.

### Step 15: Access Your Service
```bash
# Get the external IP/port
kubectl get services hello-k8s-service

# On Mac with Docker Desktop, usually localhost
curl http://localhost/
# Or visit in browser
```
**Why:** Test that your microservice is accessible and load balancing works.

### Step 16: See Load Balancing in Action
```bash
# Make multiple requests to see different hostnames
for i in {1..10}; do curl http://localhost/ | grep hostname; done
```
**Why:** Demonstrates that requests are distributed across your 3 pod replicas.

## Useful Commands for Management

### Scaling Your Service
```bash
# Scale to 5 replicas
kubectl scale deployment hello-k8s-deployment --replicas=5
```

### Viewing Logs
```bash
# See logs from all pods
kubectl logs -l app=hello-k8s --tail=50
```

### Cleanup
```bash
# Remove everything
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
```

## What You Just Built

You've created a complete microservice with:
- **Containerization:** App packaged with Docker
- **High Availability:** 3 replicas running simultaneously  
- **Load Balancing:** Traffic distributed across pods
- **Service Discovery:** Internal Kubernetes networking
- **External Access:** Service exposed to outside world

This is the foundation of microservices architecture - you can now create multiple services that communicate with each other!
