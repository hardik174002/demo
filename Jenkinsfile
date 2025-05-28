pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
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
  }

  post {
    success {
      echo '✅ Build and archive successful!'
    }
    failure {
      echo '❌ Build failed.'
    }
  }
}
