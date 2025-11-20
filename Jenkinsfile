pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube-Local'
        SCANNER = 'SonarQube-Scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    script {
                        def sonarScannerHome = tool "${SCANNER}"
                        sh """
                            ${sonarScannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=juice-shop \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONAR_AUTH_TOKEN
                        """
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
