pipeline {
    agent any

    environment {
        IMAGE_NAME = "nodejs-devops"
        DOCKERHUB_REPO = "akilaraamana/nodejs-devops"
        SONAR_KEY = credentials('sonarkey')  // Updated token
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
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('sonarqube-server') {
                        bat "${scannerHome}\\bin\\sonar-scanner -Dsonar.login=%SONAR_KEY%"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 30, unit: 'SECONDS') {   // Changed to 30 seconds
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKERHUB_REPO%:latest ."
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
