pipeline {
    agent any
    environment {
        SONARQUBE = 'sonarqube'
        SCANNER = 'SonarScanner'
        SONAR_PROJECT_KEY = 'hello-python'
        SONAR_API_TOKEN = credentials('sonar-token')  // Jenkins secret
        SONAR_HOST_URL = 'http://<JENKINS_VM_IP>:9000'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/<YOUR_USERNAME>/hello-python.git'
            }
        }
        stage('Install & Run Tests') {
            steps {
                sh '''
                  python3 -m pip install --upgrade pip
                  pip3 install -r requirements.txt
                  pytest -q
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    withEnv(["PATH+SONAR=${tool SCANNER}/bin"]) {
                        sh '''
                          sonar-scanner \
                            -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONAR_API_TOKEN \
                            -Dsonar.python.version=3.10
                        '''
                    }
                }
            }
        }
        stage('Deploy to App VM') {
            steps {
                sshagent(credentials: ['gce-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no <app-username>@<APP_VM_IP> "mkdir -p ~/app"
                        scp -o StrictHostKeyChecking=no -r * <app-username>@<APP_VM_IP>:~/app/
                        ssh -o StrictHostKeyChecking=no <app-username>@<APP_VM_IP> "
                          nohup python3 ~/app/app.py > ~/app/app.log 2>&1 &
                        "
                    '''
                }
            }
        }
    }
}
