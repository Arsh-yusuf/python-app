pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('DOCKER_CREDS')
        SONARQUBE = 'SonarQube'
        IMAGE_NAME = "asus/python-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Arsh-yusuf/python-app.git'
            }
        }

        stage('Gitleaks Scan') {
            steps {
                bat 'docker run --rm -v "$PWD:/repo" zricethezav/gitleaks:latest detect --source /repo --no-banner --redact'
            }
        }

        stage('Build') {
            steps {
                bat 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat '''
                        sonar-scanner \
                          -Dsonar.projectKey=python-app \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                bat 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image ${IMAGE_NAME}:latest'
            }
        }

        stage('Push Image') {
            steps {
                bat '''
                    echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['SSH_KEY']) {
                    bat '''
                        ssh -o StrictHostKeyChecking=no ec2-user@<your-server-ip> '
                            cd ~/app &&
                            docker compose down &&
                            docker compose pull &&
                            docker compose up -d
                        '
                    '''
                }
            }
        }
    }
}


