pipeline {
  agent {
    docker { image 'node:20' } // Node.js + npm sudah ada di image ini
  }

  environment {
    IMAGE_NAME = 'wilsonnn06/enduser-app'
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install deps & Test') {
      steps {
        sh 'npm ci'
        sh 'npm test'
      }
    }

    stage('Build') {
      steps { sh 'npm run build' }
    }

    stage('Build Docker Image') {
      steps {
        // perhatikan: image 'node:20' tidak punya docker cli
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
