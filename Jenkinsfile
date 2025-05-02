pipeline {
    agent any  // This defines a default agent

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Docker Hub credentials
        SONAR_TOKEN = credentials('sonarqube-token')             // SonarQube credentials
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
                sh 'docker build -t mahit781/secure-cicd-pipeline:latest .'
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy security scan...'
                sh 'trivy image mahit781/secure-cicd-pipeline:latest'
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push mahit781/secure-cicd-pipeline:latest
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
            cleanWs()
        }
    }
}
   
       
