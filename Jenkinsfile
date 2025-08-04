
pipeline {
    agent none  // Define agents per stage

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yourusername/flask-app.git'
            }
        }

        stage('Install & Test') {
            agent {
                docker {
                    image 'python:3.9-slim'
                    args '-v /tmp/pip-cache:/root/.cache/pip'  # Cache pip packages
                }
            }
            steps {
                sh 'pip install -r requirements.txt'
                sh 'python -m pytest tests/'  # Run unit tests
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

        stage('Push to Docker Hub') {
            agent {
                docker {
                    image 'docker:20.10.17'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            environment {
                DOCKER_HUB_CREDS = credentials('docker-hub-credentials')
            }
            steps {
                script {
                    sh "echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin"
                    docker.withRegistry('https://index.docker.io/v1/') {
                        docker.image("flask-app:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy') {
            agent any
            steps {
                sh 'docker run -d -p 5000:5000 flask-app:${env.BUILD_ID}'
            }
        }
    }

    post {
        always {
            cleanWs()  # Clean workspace
        }
        success {
            slackSend(color: 'good', message: "✅ Build ${env.BUILD_NUMBER} succeeded!")
        }
        failure {
            slackSend(color: 'danger', message: "❌ Build ${env.BUILD_NUMBER} failed!")
        }
    }
}
