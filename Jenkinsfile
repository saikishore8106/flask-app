pipeline {
    agent any  // Changed from 'none' to avoid node label requirements
    
    environment {
        DOCKER_IMAGE = "flask-app:${env.BUILD_ID}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/saikishore8106/flask-app.git'
                    ]]
                ])
            }
        }

        stage('Install & Test') {
            agent {
                docker {
                    image 'python:3.9-slim'
                    args '-v /tmp/pip-cache:/root/.cache/pip'
                    reuseNode true
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
                    reuseNode true
                }
            }
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push to Docker Hub') {
            when {
                branch 'main'
            }
            agent {
                docker {
                    image 'docker:20.10.17'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
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
