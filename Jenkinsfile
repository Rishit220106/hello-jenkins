pipeline {
    agent any

    environment {
        // Reference the SonarQube server configured in Jenkins
        SONARQUBE_SERVER = 'SonarQube-Docker'
    }

    tools {
        // Reference the SonarScanner tool configured in Jenkins
        tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'

    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout your repo
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Python dependencies
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                // Run Python tests
                sh '. venv/bin/activate && pytest tests'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                // This tells the pipeline which server configuration to use
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                // Run SonarQube scanner
                sh """
                . venv/bin/activate
                sonar-scanner \
                  -Dsonar.projectKey=hello-python \
                  -Dsonar.sources=. \
                  -Dsonar.host.url=http://34.61.69.21:9000 \
                  -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }
    }

    post {
        always {
            // Archive test results and reports if needed
            junit 'tests/*.xml'
        }
    }
}
