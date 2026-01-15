pipeline {
    agent any

    environment {
        IMAGE_NAME = 'vite-app'
        CONTAINER_NAME = 'vite-container'
        PORT = '8081'
        BUILD_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Verify Docker') {
            steps {
                sh 'docker --version'
                sh 'docker ps'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build --no-cache -t ${IMAGE_NAME}:${BUILD_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    sh """
                        if docker ps -q -f name=${CONTAINER_NAME}; then
                            echo 'Stopping existing container...'
                            docker stop ${CONTAINER_NAME}
                            sleep 2
                        fi
                    """
                    
                    sh """
                        if docker ps -aq -f name=${CONTAINER_NAME}; then
                            echo 'Removing existing container...'
                            docker rm ${CONTAINER_NAME}
                        fi
                    """
                    
                    sh "docker run -d -p ${PORT}:80 --name ${CONTAINER_NAME} ${IMAGE_NAME}:${BUILD_TAG}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sleep 5                    
                    sh "docker ps -f name=${CONTAINER_NAME} | grep ${CONTAINER_NAME}"
                    sh "curl -f http://localhost:${PORT} || exit 1"
                }
            }
        }

        stage('Cleanup Old Images') {
            steps {
                sh """
                    docker image prune -f
                    docker images ${IMAGE_NAME} --format '{{.Tag}}' | grep -v 'latest' | grep -v '${BUILD_TAG}' | head -n -2 | xargs -r docker rmi ${IMAGE_NAME}:  || true
                """
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed!  Check logs for details.'
        }
    }
}
