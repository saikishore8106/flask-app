pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "flask-app:${env.BUILD_ID}"
        PIP_CACHE_DIR = '/tmp/pip-cache'
        // Use compatible versions
        FLASK_VERSION = '2.2.5'
        WERKZEUG_VERSION = '2.2.3'
        PYTEST_VERSION = '7.4.0'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            agent {
                docker {
                    image 'python:3.9-slim'
                    args '-v $PIP_CACHE_DIR:/root/.cache/pip --user 0'
                    reuseNode true
                }
            }
            steps {
                sh """
                    pip install --user flask==${FLASK_VERSION} werkzeug==${WERKZEUG_VERSION} pytest==${PYTEST_VERSION}
                    python -m pytest tests/
                """
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
