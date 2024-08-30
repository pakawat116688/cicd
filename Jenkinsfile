pipeline {
    agent any

    environment {
        // Global environment variable for image tag
        IMAGE_TAG = 'gb1' // Change this to your desired tag
        NEXUS_URL = 'http://192.168.1.215:9876'
        IMAGE_NAME = '192.168.1.215:9876/go/gofiber'
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Clone the Git repository
                git 'https://github.com/pakawat116688/cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"'
                    // Build Docker image with tag
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh 'echo "Push Docker image url: ${NEXUS_URL}"'
                    // Login to Nexus registry
                    docker.withRegistry("${NEXUS_URL}", 'nexus-credentials') {
                        // Push Docker image to Nexus
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push("${IMAGE_TAG}")
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up
            cleanWs()
        }
    }
}
