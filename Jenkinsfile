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

        // SAST with SonarQube
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

        // SCA with OWASP Dependency Check
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

        // SCA with Retire.js for JavaScript vulnerabilities
        stage('JS Vulnerability Scan') {
            steps {
                // Install Retire.js globally or locally as a project dependency
                sh 'npm install -g retire.js' 

                // Run the scan. The '--outputformat json' can be useful for report parsing.
                // The '--path .' scans the current directory.
                sh 'retire --js --path . --exitwith 0'
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
