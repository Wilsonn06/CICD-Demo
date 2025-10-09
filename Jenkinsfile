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
    image: docker:25.0
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
            defaultContainer 'node'
        }
    }

    environment {
        DOCKER_IMAGE = "wilsonnn06/cicd-demo:${BUILD_NUMBER}"
        CONFIG_REPO = "https://github.com/Wilsonn06/CICD-Demo-Config.git"
        CONFIG_PATH = "CICD-Demo-Config/dev/deployment.yaml"
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
                    sh 'chmod +x node_modules/.bin/mocha'
                    sh 'npx mocha'
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
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-cred', 
                        usernameVariable: 'DOCKERHUB_USERNAME', 
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )]) {
                        sh '''
                            echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                            docker push ${DOCKER_IMAGE}
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Update K8s Manifest in Git') {
            steps {
                container('node') {
                    withCredentials([usernamePassword(
                        credentialsId: 'git-cred', 
                        usernameVariable: 'GIT_USERNAME', 
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh '''
                            rm -rf CICD-Demo-Config
                            git clone https://$GIT_USERNAME:$GIT_TOKEN@github.com/Wilsonn06/CICD-Demo-Config.git
                            cd CICD-Demo-Config/dev

                            # Update tag image di deployment.yaml
                            sed -i "s|image: wilsonnn06/cicd-demo:.*|image: ${DOCKER_IMAGE}|" deployment.yaml

                            git config --global user.email "jenkins@ci.local"
                            git config --global user.name "Jenkins CI"
                            git add deployment.yaml
                            git commit -m "Update image tag to ${DOCKER_IMAGE}"
                            git push origin main
                        '''
                    }
                }
            }
        }
    }
}
