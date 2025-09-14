pipeline {
    agent any

    tools {
        nodejs 'node' // Назва з Global Tool Configuration
    }

    environment {
        // Визначаємо порт та назву образу в залежності від гілки
        APP_PORT = (env.BRANCH_NAME == 'main') ? '3000' : '3001'
        DOCKER_IMAGE_NAME = (env.BRANCH_NAME == 'main') ? 'nodemain' : 'nodedev'
        DOCKER_IMAGE_TAG = 'v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building NodeJS application..."
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Running NodeJS tests..."
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Зупиняємо і видаляємо старий контейнер, якщо він існує
                    def containerId = sh(script: "docker ps -q --filter name=${DOCKER_IMAGE_NAME}", returnStdout: true).trim()
                    if (containerId) {
                        echo "Stopping and removing old container ${containerId}"
                        sh "docker stop ${containerId}"
                        sh "docker rm ${containerId}"
                    }

                    echo "Deploying ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} on port ${APP_PORT}"
                    // Запускаємо новий контейнер
                    sh "docker run -d --name ${DOCKER_IMAGE_NAME} -p ${APP_PORT}:3000 ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
