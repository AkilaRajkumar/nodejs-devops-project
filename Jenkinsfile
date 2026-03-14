pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonarkey')        
        DOCKER_HUB_CREDENTIALS = 'dockerhub-token' 
        IMAGE_NAME = 'akilaraamana/nodejs-devops-project'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/AkilaRajkumar/nodejs-devops-project.git', credentialsId: 'github-token'
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
                SONAR_SCANNER_HOME = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    bat """
                    %SONAR_SCANNER_HOME%\\bin\\sonar-scanner.bat -Dsonar.login=%SONAR_TOKEN%
                    """
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %IMAGE_NAME%:%IMAGE_TAG% ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push %IMAGE_NAME%:%IMAGE_TAG%"
            }
        }

        stage('Deploy Container') {
            steps {
                bat """
                docker rm -f nodejs-devops || exit 0
                docker run -d --name nodejs-devops -p 8080:8080 %IMAGE_NAME%:%IMAGE_TAG%
                """
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
