pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Credentials for Docker Hub
    SONAR_TOKEN = credentials('sonarqube-token')             // Credentials for SonarQube
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/Mahit781/secure-cicd-pipeline.git'
      }
    }

    stage('Gitleaks Scan') {
      steps {
        sh 'gitleaks detect --source . --verbose --redact'
      }
    }

    stage('Static Code Analysis') {
      steps {
        withSonarQubeEnv('MySonarQube') {
          sh 'sonar-scanner'
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t your-image-name .'
      }
    }

    stage('Trivy Scan') {
      steps {
        sh 'trivy image your-image-name'
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push your-image-name
          '''
        }
      }
    }

    stage('Deploy to VM') {
      steps {
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
      cleanWs()  // Clean up workspace after the build
    }
  }
}
