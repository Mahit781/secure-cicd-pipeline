pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'mahit781/secure-cicd-pipeline:latest'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    
    stages {
        stage('Checkout SCM') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Gitleaks Scan') {
            steps {
                echo 'Running Gitleaks scan...'
                bat 'C:\\tools\\gitleaks\\gitleaks.exe detect --source . --verbose --redact'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy security scan...'
                script {
                    // Fix volume mount issue for Windows
                    sh 'docker run --rm -v //var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_IMAGE}'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                echo 'Deploying Docker image to VM...'
                // Add your VM deployment steps here
            }
        }
    }

    post {
        always {
            node {
                echo 'Cleaning workspace...'
                cleanWs()
            }
        }
    }
}
