pipeline {
    agent any

    environment {
        IMAGE_NAME = 'holiday-events'
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_PORT = '3000'
        CONTAINER_NAME = 'holiday-events-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test & Health Logic') {
            steps {
                sh 'node --check server.js'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true
                docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:3000 --restart unless-stopped ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Health Check') {
            steps {
                sh "docker ps | grep ${CONTAINER_NAME}"
            }
        }
    }
}