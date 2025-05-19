pipeline {
    agent any

    environment {
        REGISTRY = "localhost:5000"
        IMAGE_NAME = "node-hello-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = "node-hello-app"
        GIT_REPO = 'https://github.com/shafiurrahmanenosis/node-express-hello-devfile-no-dockerfile.git'
        BRANCH = 'main'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("http://${REGISTRY}") {
                        docker.image("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Run Container from Image') {
            steps {
                script {
                    // Stop and remove existing container if exists
                    sh """
                    docker rm -f ${CONTAINER_NAME} || true
                    docker pull ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker run -d --name ${CONTAINER_NAME} -p 3000:8080 ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}

