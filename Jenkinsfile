pipeline {
    agent any
    environment {
        // SonarQube configuration
        SONARQUBE = 'sonarqube'                       // SonarQube server name in Jenkins
        SCANNER = 'SonarScanner'                      // SonarScanner tool name in Jenkins
        SONAR_PROJECT_KEY = 'hello-python'
        SONAR_API_TOKEN = credentials('sonar-token') // Jenkins Secret Text
        SONAR_HOST_URL = 'http://34.63.72.79:9000'

        // GitHub repository (SSH)
        GIT_REPO = 'https://github.com/Rishit220106/hello-python.git'
        GIT_BRANCH = 'main'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}",
                    url: "https://github.com/Rishit220106/hello-python.git",
                    credentialsId: 'github-cred' // Jenkins SSH credential for GitHub
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
                        ssh -o StrictHostKeyChecking=no rishit220106@34.46.215.203 "mkdir -p ~/app"
                        scp -o StrictHostKeyChecking=no -r * rishit220106@34.46.215.203:~/app/
                        ssh -o StrictHostKeyChecking=no rishit220106@34.46.215.203 "
                          nohup python3 ~/app/app.py > ~/app/app.log 2>&1 &
                        "
                    '''
                }
            }
        }
    }
}
