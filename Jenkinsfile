pipeline {
  agent any

  environment {
    IMAGE_NAME = 'wilsonnn06/enduser-app'
    IMAGE_TAG = "${env.BUILD_NUMBER}" // tag unik tiap build
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install deps & Test') {
      steps {
        sh 'npm ci'
        sh 'npm test'
      }
    }

    stage('Build frontend/backend') {
      steps {
        sh 'npm run build'
      }
    }

    stage('Build Docker Image') {
      steps {
        // pastikan docker cli tersedia di agent ini
        sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh """
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push $IMAGE_NAME:$IMAGE_TAG
            docker logout
          """
        }
      }
    }
  }
}
