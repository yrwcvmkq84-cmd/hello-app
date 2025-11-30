pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'yrwcvmkq84-docker'
        IMAGE_NAME     = "${DOCKERHUB_USER}/hello-app"
        EC2_HOST       = 'ubuntu@ec2-98-80-96-74.compute-1.amazonaws.com'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  /usr/local/bin/docker build -t ${IMAGE_NAME}:${BUILD_ID} .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | /usr/local/bin/docker login -u "$DOCKER_USER" --password-stdin
                      /usr/local/bin/docker push ${IMAGE_NAME}:${BUILD_ID}
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-creds']) {
                    sh """
                      ssh -o StrictHostKeyChecking=no ${EC2_HOST} \\
                        "docker login -u ${DOCKERHUB_USER} && \\
                         docker pull ${IMAGE_NAME}:${BUILD_ID} && \\
                         docker stop hello-app || true && \\
                         docker rm hello-app || true && \\
                         docker run -d --name hello-app -p 8080:80 ${IMAGE_NAME}:${BUILD_ID}"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployed successfully to http://${EC2_HOST.split('@')[1]}:8080/"
        }
    }
}

