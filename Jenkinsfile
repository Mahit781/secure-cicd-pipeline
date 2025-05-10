pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        // SONAR_TOKEN = credentials('sonarqube-token') // Not needed if SonarQube is disabled
    }

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

        /*
        stage('Static Code Analysis') {
            steps {
                echo 'Running Static Code Analysis with SonarQube...'
                withSonarQubeEnv('MySonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }
        */

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy filesystem scan...'
                sh 'docker run --rm -v $PWD:/app -w /app aquasec/trivy fs .'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t mahit781/secure-cicd-pipeline:latest .'
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
