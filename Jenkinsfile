pipeline {
  agent any

  environment {
    IMAGE_NAME = "my-spring-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Lint Check') {
      steps {
        sh 'mvn checkstyle:check' // Java
      }
    }

    stage('Build JAR') {
      steps {
        echo '🔨 Building the project with Maven...'
        sh 'mvn clean install'
      }
    }

    stage('Archive Artifact') {
      steps {
        echo '📦 Archiving target/*.jar'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Prepare JAR for Docker') {
      steps {
        echo '🗃️ Preparing JAR for Docker build context...'
        sh 'cp target/*.jar app.jar'
      }
    }

    stage('Build Docker Image') {
      steps {
        echo '🐳 Building Docker image...'
        sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
      }
    }

    stage('Run Docker Container') {
      steps {
        echo '🚀 Running Docker container...'
        sh '''
          docker stop my-spring-app || true
          docker rm my-spring-app || true
          docker run -d --name my-spring-app -p 8082:8082 ${IMAGE_NAME}:${IMAGE_TAG}
        '''
      }
    }
  }

  post {
    failure {
      mail(
        to: 'hvhardik@gmail.com',
        subject: "❌ Build Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
Hi Team,

The build *${env.JOB_NAME}* #${env.BUILD_NUMBER} has **FAILED**.

Check console output at: ${env.BUILD_URL}

Regards,
Jenkins
"""
      )
    }
  }
}
