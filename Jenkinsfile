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
        /*
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
                    waitForQualityGate abortPipeline: false
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
                nodejs('NodeJS') {
                    // Install Retire.js globally
                    sh 'npm install -g retire'

                    // Run the scan on the current directory.
                    sh 'retire --path . --outputformat json --exitwith 0'                    
                }
            }
        }

        
        // Deploy application (replace with your actual deployment process)
        stage('Deploy Application') {
            steps {
                script {
                    // Install dependencies (only if not already installed)
                    //sh 'npm install'

                    // Start the Node.js app
                    sh 'node app.ts'

                    // Alternatively, if using TypeScript directly:
                    // sh 'ts-node app.ts'
                }
            }
        }
        */

        stage('OWASP ZAP Scan') {
            steps {
                script {
                    // Use ZAP plugin's zapAttack step
                    zapAttack(
                        zapPath: '/usr/local/bin/zap',  // Specify ZAP executable path (if using local install)
                        targetURL: 'https://juice-shop.herokuapp.com',  // URL of the application to be scanned
                        startUrl: 'https://juice-shop.herokuapp.com',  // Starting point for ZAP to scan
                        scanType: 'full',  // Full scan (or 'quick' for lighter scan)
                        failBuildOnAlert: true,  // Fail the build if any alerts are found
                        alertThreshold: 'Medium',  // Alert severity threshold
                        attackMode: 'full'  // Full scan or quick scan
                    )
                }
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
