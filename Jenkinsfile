pipeline {
  agent any

  environment {
    IMAGE = "gcr.io/demo-project/sync-service"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t $IMAGE:$BUILD_NUMBER ."
      }
    }

    stage('Push Image') {
      when {
        branch 'develop|staging|main'
      }
      steps {
        sh "docker push $IMAGE:$BUILD_NUMBER"
      }
    }

    stage('Deploy') {
      steps {
        script {
          if (env.BRANCH_NAME == 'develop') {
            sh 'echo Deploying to QA'
          } else if (env.BRANCH_NAME == 'staging') {
            sh 'echo Deploying to Staging'
          } else if (env.BRANCH_NAME == 'main') {
            input message: "Approve production deployment?"
            sh 'echo Deploying to Production'
          }
        }
      }
    }
  }
}
