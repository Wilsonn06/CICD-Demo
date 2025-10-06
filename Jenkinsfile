pipeline {
  agent any
  stages{
    stage("Checkout"){
      steps{
        checkout scm
      }
    }

    stage("Test"){
      steps{
        sh 'sudo apt install npm'
        sh 'npm test'
      }
    }

    stage("Build"){
      steps{
        sh 'npm run build'
      }
    }

    stage("Build Image"){
      steps{
        sh 'build docker -t CICD-Demo:1.0 .'
  }
}
