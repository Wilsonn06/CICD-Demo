pipeline {
  agent {
    kubernetes {
      label 'nodejs-docker'
      defaultContainer 'node'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: node:20
    command:
    - cat
    tty: true
  - name: docker
    image: docker:25
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
    }
  }

  environment {
    FRONTEND_DIR = 'frontend'
    IMAGE_NAME = 'wilsonnn06/CICD-Demo'
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install & Build') {
      steps {
        container('node') {
          // Install backend dependencies
          sh 'npm ci'

          // Build frontend
          dir("${env.FRONTEND_DIR}") {
            sh 'npm ci'
            sh 'npm run build'
          }

          // Prepare distribution folder
          sh '''
            set -e
            rm -rf dist || true
            mkdir -p dist
            cp -r frontend/dist dist/public
          '''
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        container('docker') {
          sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        container('docker') {
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

  post {
    always {
      archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
    }
  }
}
