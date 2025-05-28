pipeline{

    agent any

    stages{
        stage('Checkout'){
            steps{
                checkout scm
            }
        }
        stage{
            steps{
                echo "Building Jar file"
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Archive Artifact') {
          steps {
            echo '📦 Archiving built JAR...'
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
          }
        }
    }
    post {
        success {
          echo '✅ Build successful. Artifact is archived.'
        }
        failure {
          echo '❌ Build failed. Check the logs.'
        }
    }

}