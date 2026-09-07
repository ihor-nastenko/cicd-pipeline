pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        IMAGE_TAG = 'v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.IMAGE_NAME = 'nodemain'
                        env.CONTAINER_NAME = 'nodemain'
                        env.HOST_PORT = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.IMAGE_NAME = 'nodedev'
                        env.CONTAINER_NAME = 'nodedev'
                        env.HOST_PORT = '3001'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:3000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }
}
