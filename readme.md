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

## 3. Jenkinsfile Explaination
This Jenkinsfile defines a CI/CD pipeline that automates building, pushing, and verifying a Docker image. Here's a breakdown of each component:
### 1. Pipeline Structure
```bash
pipeline {
    agent any  // Runs on any available Jenkins agent
    environment {
        REGISTRY = "localhost:5000"  // Docker registry address
        IMAGE_NAME = "node-hello-app"  // Your application name
        IMAGE_TAG = "latest"  // Image tag version
    }
    // Stages and post-build actions follow...
}
```
### 2. Pipeline Stages
#### Stage 1: Checkout Code
- Pulls the latest code from your version control system.
```bash
stage('Checkout') {
    steps {
        checkout scm  // Checks out source code from SCM (Git, etc.)
    }
}
```

#### Stage 2: Build Docker Image
- Builds a Docker image from the Dockerfile in your repository.
- Tags it with: localhost:5000/node-hello-app:latest
```bash
stage('Build Docker Image') {
    steps {
        script {
            docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
        }
    }
}
```
#### Stage 3: Push to Registry
- Pushes the built image to your local Docker registry (port 5000).
```bash
stage('Push Image') {
    steps {
        script {
            docker.withRegistry("http://${REGISTRY}", '') { 
                docker.image("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
            }
        }
    }
}
```
#### Stage 4: Run for Verification
- Cleans up any existing container (ignores errors if none exists).
- Runs a new container from the pushed image, mapping port 8080.
```bash
stage('Run Image for Verification') {
    steps {
        script {
            catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                sh "docker rm -f ${IMAGE_NAME}"  // Remove old container if exists
            }
            sh "docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
        }
    }
}
```
### Post-Build Actions
- Success/Failure: Provides pipeline status notifications.
```bash
post {
    success {
        echo 'Pipeline completed successfully!'
    }
    failure {
        echo 'Pipeline failed. Check logs!'
    }
}
```

## 4 Common Issues & Fixes
### Issue 1: "docker: not found"
```bash
# Install Docker CLI in container
docker exec -it -u root jenkins \
  apt-get update && apt-get install -y docker.io
```


### Issue 2: Permission denied on docker.sock
```bash
docker run ... --group-add $(stat -c "%g" /var/run/docker.sock) ...
```
