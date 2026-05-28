pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'url-shortener'
        DOCKER_TAG = "${BUILD_NUMBER}"
        REGISTRY = 'rk936503'
    }

    tools {
        nodejs 'NodeJS-22'    
    }

    stages {
        stage('Checkout') {
            steps {
                echo ' Pulling source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo ' Installing packages...'
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo ' Running tests...'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo ' Building Docker image...'
                sh "docker build -t ${REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} ${REGISTRY}/${DOCKER_IMAGE}:latest"
            }
        }

        stage('Push to Registry') {
            steps {
                echo ' Pushing to Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${REGISTRY}/${DOCKER_IMAGE}:latest"
                }
            }
        }

                stage('Deploy to EC2') {
            steps {
                echo ' Deploying to production...'
                sshagent(['ec2-server-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@16.170.211.69 '
                            docker pull ${REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker stop url-shortener || true
                            docker rm url-shortener || true
                            docker run -d --name url-shortener \
                                -p 8000:8000 \
                                --restart always \
                                ${REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        '
                    """
                }
            }
        }

    }

    post {
        success { echo '✅ Pipeline completed successfully!' }
        failure { echo '❌ Pipeline failed!' }
    }
}