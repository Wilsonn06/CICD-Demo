pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: cicd-demo
spec:
  containers:
  - name: node
    image: node:20
    command:
    - cat
    tty: true
  - name: docker
    image: docker:25.0.2-dind
    securityContext:
      privileged: true
    tty: true
"""
            defaultContainer 'node'
        }
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            steps {
                container('node') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Build') {
            steps {
                container('node') {
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh 'docker version'
                    sh 'docker build -t cicd-demo:1.0 .'
                }
            }
        }
    }
}
