pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://192.168.56.1:9000'
        SONAR_TOKEN = 'your_sonarqube_token'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/AkilaRajkumar/nodejs-devops-project.git', branch: 'main'
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
                    bat "C:\\ProgramData\\Jenkins\\.jenkins\\tools\\hudson.plugins.sonar.SonarRunnerInstallation\\sonar-scanner\\bin\\sonar-scanner -Dsonar.token=${SONAR_TOKEN}"
                }
            }
        }

        // Skip Quality Gate check (optional)
        stage('Docker Build') {
            steps {
                bat 'docker build -t my-node-app .'
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push my-node-app'
            }
        }

        stage('Deploy Container') {
            steps {
                bat 'docker run -d -p 8080:8080 my-node-app'
            }
        }
    }
}
