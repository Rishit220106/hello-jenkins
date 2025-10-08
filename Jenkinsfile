pipeline {
    agent any

    environment {
        // SonarQube token from Jenkins credentials
        SONAR_TOKEN = credentials('sonar-token')
        PYTHONPATH = "${WORKSPACE}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rishit220106/hello-jenkins',
                    credentialsId: 'github-cred'
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh """
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                """
            }
        }

        stage('Run Tests') {
            steps {
                sh """
                    . venv/bin/activate
                    pytest --junitxml=results.xml tests
                """
            }
            post {
                always {
                    junit 'results.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Use the SonarScanner installed in Jenkins
                    def scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                    sh """
                        . venv/bin/activate
                        ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=hello-python \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://35.226.157.63:9000 \
                          -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Build, tests, and SonarQube analysis completed successfully!'
        }
        failure {
            echo 'Build failed. Check logs for errors.'
        }
    }
}
