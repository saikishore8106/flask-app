pipeline {
    agent none
    
    stages {
        stage('Checkout') {
            steps {
                node {
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
        }

        stage('Install & Test') {
            steps {
                node {
                    docker.image('python:3.9-slim').inside('-v /tmp/pip-cache:/root/.cache/pip') {
                        sh 'pip install -r requirements.txt'
                        sh 'python -m pytest tests/'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                node {
                    docker.image('docker:20.10.17').inside('-v /var/run/docker.sock:/var/run/docker.sock') {
                        script {
                            docker.build("flask-app:${env.BUILD_ID}")
                        }
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            when {
                branch 'main'
            }
            steps {
                node {
                    docker.image('docker:20.10.17').inside('-v /var/run/docker.sock:/var/run/docker.sock') {
                        withCredentials([usernamePassword(
                            credentialsId: 'docker-hub-credentials',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh """
                                echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                                docker push flask-app:${env.BUILD_ID}
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            node {
                cleanWs()
            }
        }
        success {
            node {
                slackSend(color: 'good', message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
            }
        }
        failure {
            node {
                slackSend(color: 'danger', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
            }
        }
    }
}
