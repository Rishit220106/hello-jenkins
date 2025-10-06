pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                . venv/bin/activate
                export PYTHONPATH=$PYTHONPATH:$(pwd)
                pytest --junitxml=results.xml tests
                '''
            }
        }

        stage('SonarQube Analysis') {
    steps {
        script {
            // Get the installed SonarScanner
            def scannerHome = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            
            sh """
                . venv/bin/activate
                ${scannerHome}/bin/sonar-scanner \
                  -Dsonar.projectKey=hello-python \
                  -Dsonar.sources=. \
                  -Dsonar.host.url=http://34.61.69.21:9000 \
                  -Dsonar.login=${SONAR_TOKEN}
            """
        }
    }
}

        }
    }

    post {
        always {
            junit 'results.xml'
        }
    }
}
