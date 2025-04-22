pipeline {
    agent any

    environment {
        SONARQUBE = 'sonarqube' // Name of your SonarQube config in Jenkins
        GOPATH = "${env.WORKSPACE}/go"
    }

    tools {
        go 'go_1.24' // Replace with your configured Go tool name in Jenkins
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
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=go-cicd-test \
                        -Dsonar.sources=. \
                        -Dsonar.go.coverage.reportPaths=report.out \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    """
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
            junit 'report.out' // if you have test results in JUnit format
            cleanWs()
        }
    }
}
