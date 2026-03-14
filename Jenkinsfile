pipeline {
    agent any

    environment {
        // SonarQube token, if analysis is needed
        SONAR_TOKEN = credentials('sonar-token') // replace with your Jenkins secret ID
        DOCKER_IMAGE = "akilarajkumar/nodejs-devops-project"
        DOCKER_TAG = "latest"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/AkilaRajkumar/nodejs-devops-project.git',
                    credentialsId: 'github-token'
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
            environment {
                // Inject SonarQube environment
                SONAR_HOST_URL = 'http://192.168.56.1:9000'
            }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    bat "C:\\ProgramData\\Jenkins\\.jenkins\\tools\\hudson.plugins.sonar.SonarRunnerInstallation\\sonar-scanner\\bin\\sonar-scanner -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push %DOCKER_IMAGE%:%DOCKER_TAG%"
            }
        }

        stage('Deploy Container') {
            steps {
                // Stop old container if exists
                bat "docker rm -f nodejs-devops || echo 'No existing container'"
                // Run new container
                bat "docker run -d --name nodejs-devops -p 8080:8080 %DOCKER_IMAGE%:%DOCKER_TAG%"
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
