pipeline {
    agent any
    
    environment {
        // Your Docker Hub username/organization and image name
        // Format: 'username/image-name' or 'organization/image-name'
        DOCKER_IMAGE = 'vrspi/flask-app'
        
        // Build number for versioning
        DOCKER_TAG = "${BUILD_NUMBER}"
        
        // Credential ID configured in Jenkins
        // Must match EXACTLY with the credential ID you created:
        // - Username: Your Docker Hub username
        // - Password: Your Docker Hub Access Token (starts with 'dckr_pat_')
        // - ID: 'docker-hub-credentials'
        DOCKER_CREDENTIALS = 'docker-hub-credentials'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies and Test') {
            agent {
                docker {
                    image 'python:3.9-slim'
                    reuseNode true
                }
            }
            steps {
                sh 'pip install -r requirements.txt'
                sh 'python -m pytest tests/'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS) {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                // Add your deployment steps here
                // For example, using SSH to deploy to a remote server:
                // sh 'ssh user@remote-server "docker pull ${DOCKER_IMAGE}:${DOCKER_TAG} && docker-compose up -d"'
                echo 'Deployment step - configure according to your server setup'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
} 