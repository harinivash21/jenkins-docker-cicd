
pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'harinivash014'
        IMAGE_NAME = 'devops-web'
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
    }

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

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )]) {
                        bat '''
                            @echo off
                            echo %DH_TOKEN% | docker login -u %DH_USER% --password-stdin
                            if errorlevel 1 exit /b 1

                            docker tag devops-web:latest %DH_USER%/devops-web:%BUILD_NUMBER%
                            if errorlevel 1 exit /b 1

                            docker tag devops-web:latest %DH_USER%/devops-web:latest
                            if errorlevel 1 exit /b 1

                            docker push %DH_USER%/devops-web:%BUILD_NUMBER%
                            if errorlevel 1 exit /b 1

                            docker push %DH_USER%/devops-web:latest
                            if errorlevel 1 exit /b 1

                            docker logout
                        '''
                    }
                }
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
