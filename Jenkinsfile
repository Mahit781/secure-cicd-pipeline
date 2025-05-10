pipeline {
    agent any
    
    // Define credentials - only access when needed to prevent early failure
    // environment {
    //     DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    // }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out repository...'
                git branch: 'main', url: 'https://github.com/Mahit781/secure-cicd-pipeline.git'
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
                bat 'docker build -t mahit781/secure-cicd-pipeline:latest .'
            }
        }
        
        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy security scan...'
                // Use bat instead of sh, and fix the volume mount syntax for Windows
                bat 'docker run --rm aquasec/trivy image mahit781/secure-cicd-pipeline:latest'
                // If you still need Docker socket access, use Windows-style paths:
                // bat 'docker run --rm -v //var/run/docker.sock:/var/run/docker.sock aquasec/trivy image mahit781/secure-cicd-pipeline:latest'
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                echo 'Pushing to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PWD', usernameVariable: 'DOCKERHUB_USER')]) {
                    bat 'echo %DOCKERHUB_PWD% | docker login -u %DOCKERHUB_USER% --password-stdin'
                    bat 'docker push mahit781/secure-cicd-pipeline:latest'
                }
            }
        }
        
        stage('Deploy to VM') {
            steps {
                echo 'Deploying to VM...'
                // Add your deployment steps here
                // For example:
                // bat 'ssh user@server "docker pull mahit781/secure-cicd-pipeline:latest && docker-compose up -d"'
            }
        }
    }
    
    post {
        always {
            node(null) {  // Ensure cleanWs runs inside a node
                echo 'Cleaning workspace...'
                cleanWs()
                bat 'docker logout || echo "Docker logout failed, but continuing"'
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
