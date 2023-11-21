pipeline {
    agent any 

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub')
        HELM_SERVER_SSH_CREDENTIALS = credentials('helm-server-ssh')
        HELM_CHART_PATH = 'ec2-user/home/testapp'
        HELM_RELEASE_NAME = 'my-release'
    }

    stages { 
        stage('Build Docker Image') {
            steps {  
                sh 'docker build -t prajnab9/myapp:$BUILD_NUMBER .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    sh "echo \${DOCKERHUB_CREDENTIALS_PSW} | docker login -u \${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push prajnab9/myapp:$BUILD_NUMBER'
            }
        }

        stage('Upgrade Helm Chart on Server') {
            steps {
                script {
                    // Assuming you have SSH key-based authentication to the Helm server
                    sshagent(['HELM_SERVER_SSH_CREDENTIALS']) {
                        // Replace 'helm-server-user' and 'helm-server-ip' with your Helm server details
                        sh "ssh ec2-user@172.16.2.135 'helm upgrade ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} --install --set image.repository=prajnab9/myapp --set image.tag=${BUILD_NUMBER}'"
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
