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
                echo "Image ready: ${REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                echo 'To deploy manually: docker pull rk936503/url-shortener:latest && docker run -d -p 8000:8000 rk936503/url-shortener:latest'
            }
        }
    }

    post {
        success { echo '✅ Pipeline completed successfully!' }
        failure { echo '❌ Pipeline failed!' }
    }
}