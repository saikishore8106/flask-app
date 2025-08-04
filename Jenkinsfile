pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "flask-app:${env.BUILD_ID}"
        PIP_CACHE_DIR = '/tmp/pip-cache'
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
            agent any  // Run directly on the host with Docker access
            steps {
                script {
                    // Ensure Jenkins has Docker permissions
                    sh 'sudo chmod 666 /var/run/docker.sock || true'
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push to Docker Hub') {
            when {
                branch 'main'
            }
            agent any  // Run directly on the host with Docker access
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
