pipeline {
    agent any  // This means the pipeline will run on any available agent

    environment {
        DOCKER_IMAGE = 'mahit781/secure-cicd-pipeline:latest'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')  // Replace this with your actual credentials ID in Jenkins
    }

    stages {
        stage('Checkout SCM') {
            steps {
                echo 'Cloning repository...'
                checkout scm  // Check out the code from your Git repository
            }
        }

        stage('Gitleaks Scan') {
            steps {
                echo 'Running Gitleaks scan...'
                bat 'C:\\tools\\gitleaks\\gitleaks.exe detect --source . --verbose --redact'  // Adjust path as necessary
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    docker.build(DOCKER_IMAGE)  // Build the Docker image
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy security scan...'
                script {
                    // Running Trivy to scan the Docker image
                    sh 'docker run --rm -v //var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_IMAGE}'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        docker.image(DOCKER_IMAGE).push()  // Push the Docker image to DockerHub
                    }
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                echo 'Deploying Docker image to VM...'
                // Your VM deployment steps should go here (e.g., using Ansible, SSH, or other tools)
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()  // Clean up the workspace after the pipeline execution
        }
    }
}
