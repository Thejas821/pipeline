pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        // Binds your Jenkins credential ID 'docker-cred' to variables:
        // DOCKER_CREDS_USR (thejas821) and DOCKER_CREDS_PSW (password)
        DOCKER_CREDS = credentials('docker-cred')
    }
    stages {
        stage('git') {
            steps {
                git branch: 'main', url: 'https://github.com/jaiswaladi246/Boardgame.git'
            }
        }
        stage('Maven build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Docker build & Push') {
            steps {
                // 1. Build and tag the image using your Docker Hub username variable
                sh "docker build -t ${DOCKER_CREDS_USR}/board:latest ."
                
                // 2. Authenticate securely via stdin (prevents password exposure in logs)
                sh "echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin"
                
                // 3. Push the image to Docker Hub
                sh "docker push ${DOCKER_CREDS_USR}/board:latest"
                
                // 4. Log out right after pushing to clear credentials from the executor
                sh "docker logout"
            }
        }
     }
}
