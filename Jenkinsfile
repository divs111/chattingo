pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhubcredentials'
        DOCKER_FRONTEND = 'divs11/frontend-chattingo:latest'
        DOCKER_BACKEND  = 'divs11/backend-chattingo:latest'
        COMPOSE_FILE    = 'docker-compose.yml'
        TRIVY_DISABLE_VEX_NOTICE = 'true' // disables VEX warning
    }

    stages {

        stage('Git Clone') {
            steps {
                echo "Cloning repository..."
                git branch: 'main', url: 'https://github.com/divs111/chattingo.git'
            }
        }

        stage('Image Build') {
            steps {
                echo "Building Docker images..."
                sh "docker build -t ${DOCKER_FRONTEND} ./frontend"
                sh "docker build -t ${DOCKER_BACKEND} ./backend"
            }
        }

        stage('Filesystem Scan') {
            steps {
                echo "Running Trivy filesystem scan..."
                // Continue pipeline even if vulnerability found
                sh '''
                trivy fs --severity HIGH,CRITICAL ./frontend || echo "Frontend has HIGH/CRITICAL vulnerabilities"
                trivy fs --severity HIGH,CRITICAL ./backend || echo "Backend has HIGH/CRITICAL vulnerabilities"
                '''
            }
        }

        stage('Image Scan') {
            steps {
                echo "Running Trivy image scan..."
                sh '''
                trivy image --severity HIGH,CRITICAL ${DOCKER_FRONTEND} || echo "Frontend image has HIGH/CRITICAL vulnerabilities"
                trivy image --severity HIGH,CRITICAL ${DOCKER_BACKEND} || echo "Backend image has HIGH/CRITICAL vulnerabilities"
                '''
            }
        }

        stage('Push to Registry') {
            steps {
                echo "Pushing images to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u $DOCKER_USER -p $DOCKER_PASS"
                    sh "docker push ${DOC

