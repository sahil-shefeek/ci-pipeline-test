pipeline {
    agent any

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build --no-cache -t vite-app .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop vite-container || echo Container not running
                docker rm vite-container || echo Container not found
                docker run -d -p 8081:80 --name vite-container vite-app
                '''
            }
        }
    }
}
