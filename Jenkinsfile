pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'vrspi/flask-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        // Docker Hub credentials must be configured in Jenkins with this ID
        DOCKER_CREDENTIALS = 'docker-hub-credentials'
    }
    
    options {
        // Add timestamps to console output
        timestamps()
        // Discard old builds
        buildDiscarder(logRotator(numToKeepStr: '5'))
        // Timeout if pipeline takes too long
        timeout(time: 1, unit: 'HOURS')
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Clean workspace before build
                cleanWs()
                checkout scm
            }
        }
        
        stage('Test') {
            steps {
                script {
                    try {
                        // Run tests in Python container
                        docker.image('python:3.9-slim').inside('--user root') {
                            sh '''
                                python -m pip install --no-cache-dir -r requirements.txt
                                python -m pytest tests/ --junitxml=test-results.xml
                            '''
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Tests failed: ${e.message}"
                    }
                }
            }
            post {
                always {
                    // Publish test results
                    junit allowEmptyResults: true, testResults: 'test-results.xml'
                }
            }
        }
        
        stage('Build Image') {
            steps {
                script {
                    try {
                        // Build with current build number and latest tag
                        docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Failed to build Docker image: ${e.message}"
                    }
                }
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    try {
                        // Push to Docker Hub
                        docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS) {
                            def appImage = docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}")
                            appImage.push()
                            appImage.push('latest')
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Failed to push Docker image: ${e.message}"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deployment step - configure according to your server setup'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
} 