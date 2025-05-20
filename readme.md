# Jenkins CI/CD with Docker

## Prerequisites
- Docker installed on host
- Ports 9090, 50000, and 5000 available

## 1. Jenkins Setup
```bash
# Run Jenkins with Docker access
docker run -d \
  --name jenkins \
  -p 9090:8080 -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --group-add $(stat -c "%g" /var/run/docker.sock) \
  jenkins/jenkins:lts
```

## 2. Local Docker Registry
```bash
# Start registry
docker run -d \
  --name registry \
  -p 5000:5000 \
  -v registry-data:/var/lib/registry \
  registry:2

# Verify
curl http://localhost:5000/v2/_catalog
```

## 3. Jenkinsfile Example
```bash
pipeline {
    agent any
    environment {
        REGISTRY = "host.docker.internal:5000"  # For Docker containers
        IMAGE_NAME = "node-hello-app"
        IMAGE_TAG = "latest"
    }
    stages {
        stage('Build') {
            steps { sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ." }
        }
        stage('Push') {
            steps { sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}" }
        }
    }
}
```

## 4 Common Issues & Fixes
# Issue 1: "docker: not found"
```bash
# Install Docker CLI in container
docker exec -it -u root jenkins \
  apt-get update && apt-get install -y docker.io
```
# Issue2: Issue 2: Permission denied on docker.sock
```bash
docker run ... --group-add $(stat -c "%g" /var/run/docker.sock) ...
```
