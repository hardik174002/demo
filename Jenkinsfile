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
        sh 'mvn clean package -DskipTests'
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
    success {
      echo '✅ Build, image, and container run successful!'
    }
    failure {
      echo '❌ Build failed.'
    }
  }
}
