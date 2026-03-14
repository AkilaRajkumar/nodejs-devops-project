pipeline {
    agent any

    environment {
        IMAGE_NAME = 'akilaraamana/nodejs-devops-project'
        IMAGE_TAG  = 'latest'
        CONTAINER_NAME = 'nodejs-devops'
        HOST_PORT = '8080'
        CONTAINER_PORT = '8080'
        SONAR_SERVER = 'SonarQube'       // Your Jenkins SonarQube server name
        SONAR_KEY = credentials('sonarkey')   // Jenkins credential ID for SonarQube token
        DOCKERHUB_TOKEN = credentials('dockerhub-token') // Jenkins credential ID for DockerHub token
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    bat "sonar-scanner -Dsonar.projectKey=nodejs-devops-project -Dsonar.sources=. -Dsonar.host.url=%SONAR_HOST_URL% -Dsonar.login=${SONAR_KEY}"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'DOCKERHUB_TOKEN', url: '') {
                    bat "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Stop Existing Container & Free Port') {
            steps {
                script {
                    // Remove existing container
                    bat "for /f %%i in ('docker ps -a -q --filter name=${CONTAINER_NAME}') do docker rm -f %%i"

                    // Kill any process using HOST_PORT
                    def portCheck = bat(script: "netstat -ano | findstr :${HOST_PORT}", returnStdout: true).trim()
                    if (portCheck) {
                        def pid = portCheck.tokenize()[-1]
                        echo "Killing process ${pid} using port ${HOST_PORT}"
                        bat "taskkill /F /PID ${pid}"
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                bat "docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:${CONTAINER_PORT} ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed! Check logs."
        }
        success {
            echo "Pipeline succeeded! Docker container running on port ${HOST_PORT}."
        }
    }
}
