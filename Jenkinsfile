pipeline {
    agent any

    environment {
        // Jenkins credential for SonarQube
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout your Git repository
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                // Set up Python virtual environment and install requirements
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
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
                // Run SonarQube analysis directly via sonar-scanner
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
        junit 'results.xml'
    }
}

    }
}
