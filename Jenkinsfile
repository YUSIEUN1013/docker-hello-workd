pipeline {
    agent {
        kubernetes {
            label 'docker-build'
            defaultContainer 'docker'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: docker-build
spec:
  containers:
  - name: git
    image: alpine/git
    command:
    - cat
    tty: true
  - name: docker
    image: docker:28.1.1
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
        DOCKERHUB_CRED = credentials('your_dockerhub_cred')  // Jenkins Credentials ID
        def appImage
    }
    stages {
        stage('Checkout') {
            steps {
                container('git') {
                    checkout scm
                }
            }
        }
        stage('Build'){
            container('docker'){
                script {
                    appImage = docker.build("selinux1/node-hello-world")
                }
            }
        }
        
        stage('Test'){
            container('docker'){
                script {
                    appImage.inside {
                        sh 'npm install'
                        sh 'npm test'
                    }
                }
            }
        }
        stage('Push') {
            steps {
                container('docker') {
                    sh """
                        echo \$DOCKERHUB_CRED_PSW | docker login -u \$DOCKERHUB_CRED_USR --password-stdin
                        docker push selinux1/node-hello-world:\$BUILD_NUMBER
                        docker tag selinux1/node-hello-world:\$BUILD_NUMBER selinux1/node-hello-world:latest
                        docker push selinux1/node-hello-world:latest
                    """
                }
            }
        }
    }
}
