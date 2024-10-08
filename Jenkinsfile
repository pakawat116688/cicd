pipeline {
    agent any

    environment {
        IMAGE_TAG = "v${BUILD_NUMBER}" // Change this to your desired tag, BUID_NUMBER is number run pipeline
    }

    stages {
        stage('Set Environment Variable') {
            steps {
                withCredentials([string(credentialsId: 'nexus-url', variable: 'VAR_NEXUS_URL')]) { 
                    script {
                        env.NEXUS_URL = "${VAR_NEXUS_URL}"
                    }
                }
                withCredentials([string(credentialsId: 'gofiber-image-name', variable: 'VAR_IMAGE')]) { 
                    script {
                        env.IMAGE_NAME =  "${VAR_IMAGE}/go/gofiber"
                    }
                }
                withCredentials([string(credentialsId: 'my-email', variable: 'VAR_EMAIL')]) { 
                    script {
                        env.EMAIL =  "${VAR_EMAIL}"
                    }
                }
            }
        }

        stage('Clone Repository') {
            steps {
                // Clone the Git repository
                git branch: 'main', url: 'https://github.com/pakawat116688/cicd.git'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                script {
                    // Create File 
                    dependencyCheck additionalArguments: '--project "cicd" --out . --format ALL --prettyPrint', 
                                    odcInstallation: 'OWASP-Dependency-Check-Vulnerabilities', 
                                    scanpath: './'
                    
                    // Show on Jenkins Dashboard
                    dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                }
            }
            post {
                always {

                    script {
                        sh "mv dependency-check-report.html dependency-check-report-${BUILD_NUMBER}.html"
                    }

                    // Archive the report as an artifact and publish HTML report
                    archiveArtifacts artifacts: "dependency-check-report-${BUILD_NUMBER}.html", allowEmptyArchive: true
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: "dependency-check-report-${BUILD_NUMBER}.html",
                        reportName: 'OWASP Dependency Check Report'
                    ])
                }
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

        stage('Deploy to Kubernetes') {
            steps {
                script {

                    sh """
                    sed -i "s|image: ''|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" k8s/deployment.yaml
                    """

                    withKubeConfig([credentialsId: 'context']) {
                        sh 'kubectl apply -f k8s/deployment.yaml'
                        sh 'kubectl apply -f k8s/service.yaml'
                        sh 'kubectl apply -f k8s/ingress.yaml'
                    }
                }
            }
            
        }

        stage('Send Email') {
            steps {
                script {
                    emailext(
                        subject: "Run CICD Pipeline OWASP Success",
                        body: "Build golong image -> push image to nexus -> Deploy to kubernetes Cluster",
                        attachmentsPattern: "dependency-check-report-${BUILD_NUMBER}.html",
                        to: "${EMAIL}"
                    )
                }
            }
        }
    }

    // Clear Workspace after run pipeline success
    post {
        always {
            echo "Pipeline Success"
            // Clean up
            // cleanWs()
        }
    }
}
