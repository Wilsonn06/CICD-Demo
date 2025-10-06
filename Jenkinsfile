pipeline {
  agent any
  stages {
    stage("Checkout") {
      steps {
        checkout scm
      }
    }

    stage("Test") {
      steps {
        sh 'npm install'    
        sh 'npm test'
      }
    }

    stage("Build") {
      steps {
        sh 'npm run build'
      }
    }

    stage("Build Image") {
      steps {
        sh 'docker build -t CICD-Demo:1.0 .'
      }
    }
  }
}
