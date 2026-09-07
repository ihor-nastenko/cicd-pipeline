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
                        env.IMAGE_NAME = 'ihor8nastenko8devops/nodemain'
                        env.DEPLOY_JOB = 'Deploy_to_main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.IMAGE_NAME = 'ihor8nastenko8devops/nodedev'
                        env.DEPLOY_JOB = 'Deploy_to_dev'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage('Trigger Deployment') {
            steps {
                script {
                    build job: env.DEPLOY_JOB,
                        parameters: [
                            string(name: 'IMAGE_TAG', value: env.IMAGE_TAG)
                        ]
                }
            }
        }
    }
}
