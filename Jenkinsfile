podTemplate(label: 'docker-build', 
  containers: [
    containerTemplate(
      name: 'git',
      image: 'alpine/git',
      command: 'cat',
      ttyEnabled: true
    ),
    containerTemplate(
      name: 'docker',
      image: 'docker',
      command: 'cat',
      ttyEnabled: true
    ),
  ],
  volumes: [ 
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'), 
  ]
) {
    node('docker-build') {
        def dockerHubCred = "your_dockerhub_cred"
        def appImage
        
        stage('Checkout'){
            container('git'){
                checkout scm
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
                  sh 'docker run --rm selinux1/node-hello-world npm test'
                }  
            }
        }

        stage('Push'){
            container('docker'){
                script {
                    docker.withRegistry('https://registry.hub.docker.com', dockerHubCred){
                        appImage.push("${env.BUILD_NUMBER}")
                        appImage.push("latest")
                    }
                }
            }
        }

        stage('Trigger CD Pipeline') {
            steps {
                build job: 'cd-pipeline', // 실제 CD 아이템 이름
                    parameters: [
                        string(name: 'IMAGE_TAG', value: imageTag)
                    ]
            }
        }
    }
    
}
