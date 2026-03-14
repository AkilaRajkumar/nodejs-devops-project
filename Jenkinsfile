pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://192.168.56.1:9000'          // SonarQube server URL
        SONAR_LOGIN = 'your_sonarqube_token'               // SonarQube token
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
                    bat "C:\\ProgramData\\Jenkins\\.jenkins\\tools\\hudson.plugins.sonar.SonarRunnerInstallation\\sonar-scanner\\bin\\sonar-scanner -Dsonar.login=${SONAR_LOGIN}"
                }
            }
        }

        stage('Check Quality Gate') {
            steps {
                script {
                    def maxWaitMinutes = 10               // max wait time
                    def startTime = System.currentTimeMillis()
                    def qgStatus = "IN_PROGRESS"

                    // Get the SonarQube task ID from the analysis report
                    def ceTaskUrl = "${SONAR_HOST_URL}/api/ce/task?component=nodejs-devops"

                    while (qgStatus == "IN_PROGRESS") {
                        echo "Checking SonarQube Quality Gate..."
                        def response = httpRequest(url: ceTaskUrl, httpMode: 'GET', authentication: 'sonar-token')
                        def json = readJSON text: response.content
                        def taskStatus = json.task.status
                        def analysisId = json.task.analysisId

                        if (taskStatus == "SUCCESS" && analysisId) {
                            def qgResponse = httpRequest(url: "${SONAR_HOST_URL}/api/qualitygates/project_status?analysisId=${analysisId}", httpMode: 'GET', authentication: 'sonar-token')
                            def qgJson = readJSON text: qgResponse.content
                            qgStatus = qgJson.projectStatus.status
                            echo "Quality Gate status: ${qgStatus}"

                            if (qgStatus == "OK") {
                                echo "Quality Gate passed ✅"
                                break
                            } else if (qgStatus == "ERROR" || qgStatus == "WARN") {
                                error "Quality Gate failed ❌: ${qgStatus}"
                            }
                        }

                        // Check timeout
                        if ((System.currentTimeMillis() - startTime) > maxWaitMinutes * 60 * 1000) {
                            error "Quality Gate check timed out ⏱"
                        }

                        sleep 10   // wait 10 seconds before polling again
                    }
                }
            }
        }

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
