pipeline {
    agent any

    environment {
        SONARQUBE = 'sonarqube' // matches Jenkins SonarQube server name
        GOPATH = "${env.WORKSPACE}/go"
    }

    tools {
        go 'go_1.24'               // Use the name configured in Jenkins
        sonarScanner 'sccli' // Matches the SonarQube Scanner name
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
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=go-cicd-test \
                        -Dsonar.sources=. \
                        -Dsonar.go.coverage.reportPaths=report.out
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
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
