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
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '''
                    -o './'
                    -s './'
                    -f 'ALL'
                    --prettyPrint
                ''', odcInstallation: 'OWASP-DepCheck'
            }
        }
    }

    post {
        always {
            // Publish the results after the scan
            dependencyCheckPublisher pattern: 'dependency-check-report.xml'
        }
    }
}
