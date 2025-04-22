pipeline {
    agent any

    environment {
        SONARQUBE = 'sonarqube' // Jenkins SonarQube name from Configure System
        GOPATH = "${env.WORKSPACE}/go"
    }

    tools {
        go 'go_1.24'  // Replace with your Go tool name in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/TakawatP/go-cicd-test.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'go mod tidy'
                sh 'go build -v ./...'
            }
        }

        stage('Test') {
            steps {
                sh 'go test -v ./... > report.out'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${env.SONARQUBE}") {
                    script {
                        scannerHome = tool name: 'SonarScannerCLI', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=go-cicd-test \
                            -Dsonar.sources=. \
                            -Dsonar.go.coverage.reportPaths=report.out
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
