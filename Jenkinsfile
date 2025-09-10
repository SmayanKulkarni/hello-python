pipeline {
    agent any

    environment {
        SONARQUBE = 'sonarqube'
        SCANNER = 'SonarScanner'
        SONAR_PROJECT_KEY = 'hello-python'
        SONAR_API_TOKEN = credentials('sonar-token')  // Jenkins secret
        SONAR_HOST_URL = 'http:// 34.45.142.77:9000' //Jenkin VM IP
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/SmayanKulkarn/hello-python.git'  //github username
            }
        }

        stage('Install & Run Tests') {
            steps {
                sh '''
                  echo "Installing dependencies..."
                  python3 -m pip install --upgrade pip
                  pip3 install --user -r requirements.txt
                  
                  echo " Running tests..."
                  python3 -m pytest -q
                '''
            }
        }

        stage('SonarQube Analysis (Async)') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    withEnv(["PATH+SONAR=${tool SCANNER}/bin"]) {
                        sh '''
                          echo "Running SonarQube scan asynchronously..."
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
                        echo "Deploying with systemd..."

                        ssh -o StrictHostKeyChecking=no smayankulkarni05@35.225.197.83"mkdir -p /home/ smayankulkarni05/app"    //appvm (to check IP address whoami)
                        scp -o StrictHostKeyChecking=no -r * smayankulkarni05@35.225.197.83:/home/ smayankulkarni05/app/

                        ssh -o StrictHostKeyChecking=no smayankulkarni05@35.225.197.83"
                          sudo systemctl daemon-reload &&
                          sudo systemctl restart flaskapp &&
                          sudo systemctl enable flaskapp
                        "

                        echo "Deployment complete. Check: curl http:// 35.225.197.83:8080"
                    '''
                }
            }
        }

        stage('Optional: Check SonarQube Quality Gate') {
            steps {
                script {
                    echo "Fetching SonarQube Quality Gate result (non-blocking)..."
                    sh """
                      curl -s -u $SONAR_API_TOKEN: \
                        "$SONAR_HOST_URL/api/qualitygates/project_status?projectKey=$SONAR_PROJECT_KEY" \
                        | jq '.projectStatus.status'
                    """
                    echo "You can manually inspect the SonarQube dashboard for full details."
                }
            }
        }
    }

    post {
        success {
            echo " Pipeline Succeeded"
        }
        failure {
            echo " Pipeline Failed"
        }
    }
}
