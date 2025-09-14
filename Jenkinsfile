pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    // Блок environment залишаємо порожнім або видаляємо
    // environment {}

    stages {
        stage('Initialize') {
            steps {
                script {
                    // Всю логіку визначення змінних переносимо сюди
                    echo "Current branch is ${env.BRANCH_NAME}"
                    if (env.BRANCH_NAME == 'main') {
                        env.APP_PORT = '3000'
                        env.DOCKER_IMAGE_NAME = 'nodemain'
                    } else {
                        env.APP_PORT = '3001'
                        env.DOCKER_IMAGE_NAME = 'nodedev'
                    }
                    env.DOCKER_IMAGE_TAG = 'v1.0'
                    echo "Deploying to port ${env.APP_PORT} with image ${env.DOCKER_IMAGE_NAME}"
                }
            }
        }

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
                echo "Building Docker image ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
                sh "docker build -t ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerId = sh(script: "docker ps -q --filter name=${env.DOCKER_IMAGE_NAME}", returnStdout: true).trim()
                    if (containerId) {
                        echo "Stopping and removing old container ${containerId}"
                        sh "docker stop ${containerId}"
                        sh "docker rm ${containerId}"
                    }

                    echo "Deploying ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} on port ${env.APP_PORT}"
                    sh "docker run -d --name ${env.DOCKER_IMAGE_NAME} -p ${env.APP_PORT}:3000 ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
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
