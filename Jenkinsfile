pipeline {
    agent any

    environment {
        IMAGE_NAME = 'holiday-events'
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_PORT = '3000'
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
                sh 'node -e "require(\'./server.js\')"'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Ansible Deploy') {
            steps {
                sh """
                ansible-playbook -i ansible/inventory.ini ansible/deploy.yml \
                    -e "image_name=${IMAGE_NAME}" \
                    -e "image_tag=${IMAGE_TAG}" \
                    -e "app_port=${APP_PORT}"
                """
            }
        }

        stage('Health Check') {
            steps {
                sh "curl --fail http://localhost:${APP_PORT}/ || exit 1"
            }
        }
    }
}