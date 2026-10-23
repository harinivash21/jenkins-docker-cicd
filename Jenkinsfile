
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                bat 'git --version'
                bat 'docker --version'
                bat 'if not exist index.html exit /b 1'
                bat 'if not exist Dockerfile exit /b 1'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t devops-web:latest .'
            }
        }

        stage('Deploy Container') {
            steps {
                bat 'docker rm -f devops-web || exit /b 0'
                bat 'docker run -d --name devops-web -p 8081:80 devops-web:latest'
            }
        }

        stage('Smoke Test') {
            steps {
                bat 'curl.exe -f http://localhost:8081'
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
