pipeline {
    agent any

    environment {
        IMAGE_NAME = 'mahit781/sample-app'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out repository...'
                git 'https://github.com/Mahit781/secure-cicd-pipeline.git'
            }
        }

        stage('Gitleaks Scan') {
            steps {
                echo 'Running Gitleaks scan...'
                bat 'C:\\tools\\gitleaks\\gitleaks.exe detect --source . --verbose --redact'
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy filesystem scan...'
                script {
                    def workspacePath = pwd().replace('\\', '/').replace('C:', '/c')
                    sh """
                        docker run --rm -v ${workspacePath}:/app -w /app aquasec/trivy fs .
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to DockerHub') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // Set correct ID in Jenkins
            }
            steps {
                echo 'Pushing image to DockerHub...'
                script {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                echo 'Deploying to VM...'
                // Replace with SSH and Docker run command as needed
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}
