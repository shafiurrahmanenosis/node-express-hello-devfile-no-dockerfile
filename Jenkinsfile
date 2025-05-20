pipeline {
    agent any

    environment {
        REGISTRY = "localhost:5000"
        IMAGE_NAME = "node-hello-app"
        IMAGE_TAG = "1"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/shafiurrahmanenosis/node-express-hello-devfile-no-dockerfile.git'
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
                    docker.withRegistry("http://${REGISTRY}", '') {
                        docker.image("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Pull and Run Image') {
            steps {
                script {
                    sh "docker pull ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker run -d -p 8080:8080 --name test-container ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            script {
                sh "docker rm -f test-container || true"
            }
        }
        failure {
            echo 'Build failed!'
        }
    }
}
