pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhubcredentials' 
        DOCKER_FRONTEND = 'divs11/frontend-chattingo:latest'
        DOCKER_BACKEND  = 'divs11/backend-chattingo:latest'
        COMPOSE_FILE    = 'docker-compose.yml'
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
                sh "trivy fs --exit-code 1 --severity HIGH,CRITICAL ./frontend"
                sh "trivy fs --exit-code 1 --severity HIGH,CRITICAL ./backend"
            }
        }

        stage('Image Scan') { 
            steps {
                echo "Running Trivy image scan..."
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_FRONTEND}"
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_BACKEND}"
            }
        }

        stage('Push to Registry') { 
            steps {
                echo "Pushing images to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u $DOCKER_USER -p $DOCKER_PASS"
                    sh "docker push ${DOCKER_FRONTEND}"
                    sh "docker push ${DOCKER_BACKEND}"
                }
            }
        }

        stage('Update Compose & Deploy') { 
            steps {
                echo "Deploying containers using docker-compose..."
                sh "docker compose -f ${COMPOSE_FILE} down"
                sh "docker compose -f ${COMPOSE_FILE} up -d --build"
            }
        }

    }

    post {
        always {
            echo "Cleaning up dangling images..."
            sh "docker image prune -f"
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}

