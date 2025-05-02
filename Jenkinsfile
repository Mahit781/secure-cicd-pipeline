pipeline {
    agent any  // This will define a default agent

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Credentials for Docker Hub
        SONAR_TOKEN = credentials('sonarqube-token')             // Credentials for SonarQube
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
                sh 'gitleaks detect --source . --verbose --redact'
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo 'Running Static Code Analysis with SonarQube...'
                withSonarQubeEnv('MySonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t yourusername/your-repo-name:latest .'  // Replace with your image name
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy security scan...'
                sh 'trivy image yourusername/your-repo-name:latest'  // Replace with your image name
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push yourusername/your-repo-name:latest  // Replace with your image name
                    '''
                }
            }
        }

        stage('Deploy to VM') {
            steps {
                echo 'Deploying to VM...'
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'my-vm',
                            transfers: [sshTransfer(sourceFiles: '**/docker-compose.yml', execCommand: 'docker-compose up -d')],
                            usePromotionTimestamp: false,
                            verbose: true
                        )
                    ]
                )
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            // Wrap cleanWs inside a node block
            node {
                cleanWs()  // Clean up workspace after the build
            }
        }
    }
}
