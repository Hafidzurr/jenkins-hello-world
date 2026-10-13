pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'hafidzurr'
        IMAGE_NAME = 'hafidzurr/jenkins-hello-world'
        APP_SERVER = '172.31.40.233'
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Server 2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'app-server-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -i "$SSH_KEY" \
                            -o StrictHostKeyChecking=yes \
                            "$SSH_USER@$APP_SERVER" \
                            "docker pull ${IMAGE_NAME}:latest &&
                             docker rm -f hello-world-app 2>/dev/null || true"

                        ssh -i "$SSH_KEY" \
                            -o StrictHostKeyChecking=yes \
                            "$SSH_USER@$APP_SERVER" \
                            "docker run -d --name hello-world-app \
                             --restart unless-stopped \
                             -p 80:3000 ${IMAGE_NAME}:latest"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'CI/CD pipeline finished.'
        }
    }
}
