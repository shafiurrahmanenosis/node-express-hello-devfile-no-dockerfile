pipeline {
    agent any

    environment {
        REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'node-hello-app'
        CONTAINER_PORT = '8080'
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }
    }
}

