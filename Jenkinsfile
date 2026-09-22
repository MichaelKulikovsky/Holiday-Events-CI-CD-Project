pipeline {
    agent any

    environment {
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'michaelkulikovsky/holiday-events'
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

        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Ansible Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-target-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                    sh """
                    ansible-playbook -i ansible/inventory.ini ansible/deploy.yml \
                        --private-key \$SSH_KEY \
                        -u \$SSH_USER \
                        -e "image_name=${IMAGE_NAME}" \
                        -e "image_tag=${IMAGE_TAG}" \
                        -e "app_port=${APP_PORT}"
                    """
                }
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                TARGET_HOST=$(grep -v '\\[\\|^$' ansible/inventory.ini | head -n 1 | awk '{print $1}')
                curl --fail http://${TARGET_HOST}:${APP_PORT}/health || exit 1
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}