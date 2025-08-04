pipeline {
    agent none

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/saikishore8106/flask-app.git'
            }
        }

        stage('Install & Test') {
            agent {
                docker {
                    image 'python:3.9-slim'
                    // Move comment outside the string:
                    args '-v /tmp/pip-cache:/root/.cache/pip'  // Cache pip packages
                }
            }
            steps {
                sh 'pip install -r requirements.txt'
                sh 'python -m pytest tests/'
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:20.10.17'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                script {
                    docker.build("flask-app:${env.BUILD_ID}")
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
