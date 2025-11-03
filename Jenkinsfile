pipeline {
    agent any

    environment {
        SONARQUBE = credentials('SONAR_TOKEN')          // SonarQube token (configured in Jenkins)
        DOCKERHUB = credentials('DOCKERHUB_CREDS') // DockerHub credentials ID
        IMAGE_NAME ="yusuf48367/python-app"  // Replace with your DockerHub repo name
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Arsh-yusuf/python-app.git'
            }
        }

        stage('Gitleaks Scan') {
            steps {
                bat '''
                echo Running Gitleaks Secret Scan...
                docker run --rm -v "%CD%:/repo" zricethezav/gitleaks:latest detect --source /repo --no-banner --redact || exit /b 0
                '''
            }
        }

        stage('Build') {
            steps {
                bat '''
                echo Installing dependencies...
                python --version
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                bat '''
                echo Running SonarQube analysis...
                sonar-scanner ^
                  -Dsonar.projectKey=python-app ^
                  -Dsonar.sources=. ^
                  -Dsonar.host.url=http://localhost:9000 ^
                  -Dsonar.login=%SONARQUBE%
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                bat '''
                echo Running Trivy Vulnerability Scan...
                docker run --rm -v "%CD%:/project" aquasec/trivy fs /project > trivy-report.txt || exit /b 0
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                bat '''
                echo Building Docker image...
                docker build -t %IMAGE_NAME%:latest .

                echo Logging in to DockerHub...
                echo %DOCKERHUB_PSW% | docker login -u %DOCKERHUB_USR% --password-stdin

                echo Pushing image to DockerHub...
                docker push %IMAGE_NAME%:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                echo Deploying with Docker Compose...
                docker-compose down || exit /b 0
                docker-compose up -d --build
                docker ps
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution complete ✅'
        }
        failure {
            echo 'Pipeline failed ❌'
        }
        success {
            echo 'Pipeline succeeded 🎉'
        }
    }
}
