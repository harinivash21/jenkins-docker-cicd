
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
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_TOKEN'
            )]) {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $env:DH_TOKEN | docker login `
                        --username $env:DH_USER `
                        --password-stdin

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker Hub login failed"
                    }

                    docker tag devops-web:latest "$env:DH_USER/devops-web:$env:BUILD_NUMBER"
                    if ($LASTEXITCODE -ne 0) { throw "Version tag failed" }

                    docker tag devops-web:latest "$env:DH_USER/devops-web:latest"
                    if ($LASTEXITCODE -ne 0) { throw "Latest tag failed" }

                    docker push "$env:DH_USER/devops-web:$env:BUILD_NUMBER"
                    if ($LASTEXITCODE -ne 0) { throw "Version push failed" }

                    docker push "$env:DH_USER/devops-web:latest"
                    if ($LASTEXITCODE -ne 0) { throw "Latest push failed" }

                    docker logout
                '''
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
