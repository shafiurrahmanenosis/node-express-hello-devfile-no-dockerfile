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
