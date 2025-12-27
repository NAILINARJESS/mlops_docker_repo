pipeline {
    agent any

    environment {
        DAGSHUB_TOKEN = credentials("dagshub-token")
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/NAILINARJESS/mlops_docker_repo.git',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Setup Environment') {
            steps {
                sh """
                python3 -m venv venv
                ./venv/bin/pip install --upgrade pip
                ./venv/bin/pip install -r requirements.txt
                """
            }
        }

        stage('ML Training & Tracking') {
            steps {
                sh """
                export DAGSHUB_TOKEN=${DAGSHUB_TOKEN}
                ./venv/bin/python src/tracking/mlflow_tracking.py
                """
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                docker build -t bot-api -f src/api/Dockerfile .
                docker build -t bot-frontend -f src/frontend/Dockerfile src/frontend
                """
            }
        }

        stage('Docker Compose Up') {
            steps {
                sh """
                docker-compose -f src/docker-compose.yml up -d --build
                """
            }
        }

        stage('Cleanup') {
            steps {
                sh """
                docker-compose -f src/docker-compose.yml down
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline CI/CD MLOps exécuté avec succès"
        }
        failure {
            echo "❌ Échec du pipeline"
        }
    }
}

