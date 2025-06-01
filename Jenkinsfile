pipeline {

  agent any

  parameters {
    string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
    choice(name: 'ENV', choices: ['dev', 'qa', 'prod'], description: 'Target deployment environment')
    booleanParam(name: 'RUN_DOCKER', defaultValue: true, description: 'Build and run Docker container?')
    booleanParam(name: 'RUN_UNIT_TEST',defaultValue:true ,description: 'Do you want to run unit test?')
    booleanParam(name:'RUN_INTEGRATION_TEST',defaultValue:true,description:'Do you want to run Integration test?')
    booleanParam(name:'RUN_LINT',defaultValue:true,description:'Do you want to run Lint?')
  }

  environment {
    IMAGE_NAME = "my-spring-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        echo "📦 Checking out branch: ${params.BRANCH}"
        sh "git checkout ${params.BRANCH}"
      }
    }

    stage('Parallel Quality Gates') {
      parallel {
        stage('Unit Tests') {
            when {
                expression { return params.RUN_UNIT_TEST }
            }
          steps {
            echo '🧪 Running unit tests...'
            sh 'mvn test -Dtest=*UnitTest'
          }
        }
        stage('Integration Tests') {
            when {
                expression { return params.RUN_INTEGRATION_TEST }
            }
          steps {
            echo '🔬 Running integration tests...'
            sh 'mvn verify -Dtest=*IntegrationTest'
          }
        }
        stage('Code Quality Check') {
            when {
                expression { return params.RUN_LINT }
            }
          steps {
            echo '🧼 Running checkstyle...'
            sh 'mvn checkstyle:check'
          }
        }
      }
    }

    stage('Build JAR') {
      steps {
        echo "🔨 Building the project for ${params.ENV} environment..."
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
      when {
        expression { return params.RUN_DOCKER }
      }
      steps {
        echo '🗃️ Preparing JAR for Docker build context...'
        sh 'cp target/*.jar app.jar'
      }
    }

    stage('Build Docker Image') {
      when {
        expression { return params.RUN_DOCKER }
      }
      steps {
        echo '🐳 Building Docker image...'
        sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
      }
    }

    stage('Run Docker Container') {
      when {
        expression { return params.RUN_DOCKER }
      }
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
    always {
      echo '🧹 [Post] Always runs'
    }
    success {
      echo '✅ [Post] Runs on SUCCESS only'
    }
    unstable {
      echo '⚠️ [Post] Runs if build is UNSTABLE'
    }
    changed {
      echo '🔁 [Post] Runs if build result changed from last time'
    }

  }
}
