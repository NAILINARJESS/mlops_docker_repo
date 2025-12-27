pipeline {
    agent any

    environment {
        DOCKER_HOST = "tcp://host.docker.internal:2375"
        IMAGE_API = "bot-api:latest"
        IMAGE_FRONTEND = "bot-frontend:latest"
        COMPOSE_FILE = "docker-compose.yml"
        DOCKERHUB_CREDENTIALS = "dockerhub-credentials" // ID de tes credentials Docker Hub
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/NAILINARJESS/mlops_docker_repo.git', branch: 'main', credentialsId: 'github-credentials'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo "Building Docker images..."
                    sh 'docker build -t $IMAGE_API -f src/api/Dockerfile .'
                    sh 'docker build -t $IMAGE_FRONTEND -f src/frontend/Dockerfile .'
                    sh 'docker images'
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker images to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: "$DOCKERHUB_CREDENTIALS",
                                                     usernameVariable: 'DOCKER_USER',
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'

                        // Tag des images pour Docker Hub
                        sh 'docker tag bot-api:latest $DOCKER_USER/bot-api:latest'
                        sh 'docker tag bot-frontend:latest $DOCKER_USER/bot-frontend:latest'

                        // Push vers Docker Hub
                        sh 'docker push $DOCKER_USER/bot-api:latest'
                        sh 'docker push $DOCKER_USER/bot-frontend:latest'
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    echo "Stopping any running containers..."
                    sh 'docker compose -f $COMPOSE_FILE down || true'

                    echo "Starting containers..."
                    sh 'docker compose -f $COMPOSE_FILE up -d --build'

                    echo "List of running containers:"
                    sh 'docker ps -a'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline terminée. Les containers tournent toujours et les images ont été poussées sur Docker Hub."
        }
    }
}

