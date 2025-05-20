pipeline {
    agent any

    environment {
        REGISTRY = "localhost:5000"
        IMAGE_NAME = "node-hello-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("http://${REGISTRY}", '') { 
                        docker.image("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Run Image for Verification') {
            steps {
                script {
                    sh "docker rm -f ${IMAGE_NAME} || true"
                    sh """
                        docker run -d \
                          --name ${IMAGE_NAME} \
                          -p 8080:8080 \
                          ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5  
                        docker logs ${IMAGE_NAME}  
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs!'
        }
    }
}
