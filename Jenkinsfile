pipeline {
    agent any

    environment {
        IMAGE_NAME = "nodejs-devops"
        DOCKERHUB_REPO = "akilaraamana/nodejs-devops"
        DOCKER_CREDENTIALS = "dockerhub-token"
    }

    tools {
        nodejs "nodejs"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/AkilaRajkumar/nodejs-devops-project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    bat 'sonar-scanner'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKERHUB_REPO%:latest .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-token',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    bat """
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker push %DOCKERHUB_REPO%:latest
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                bat """
                docker stop nodejs-container || echo Container not running
                docker rm nodejs-container || echo Container not found
                docker run -d -p 3000:3000 --name nodejs-container %DOCKERHUB_REPO%:latest
                """
            }
        }

    }
}
