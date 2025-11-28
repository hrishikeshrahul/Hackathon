pipeline {
    agent any

    environment {
        // Jenkins credentials ID that stores Docker Hub username/password
        DOCKERHUB = credentials('dockerhub-login')

        // Docker Hub username
        DOCKERHUB_USERNAME = "samanth195"

        // Docker image names
        BACKEND_IMAGE = "${DOCKERHUB_USERNAME}/mean-backend:latest"
        FRONTEND_IMAGE = "${DOCKERHUB_USERNAME}/mean-frontend:latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/samanthreddy245-debug/crud-dd-task-mean-app.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh '''
                    cd backend
                    docker build -t ${BACKEND_IMAGE} .
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                    cd frontend
                    docker build -t ${FRONTEND_IMAGE} .
                '''
            }
        }

        stage('Login to Docker Hub & Push Images') {
            steps {
                sh '''
                    echo "${DOCKERHUB_PSW}" | docker login -u "${DOCKERHUB_USR}" --password-stdin
                    docker push ${BACKEND_IMAGE}
                    docker push ${FRONTEND_IMAGE}
                '''
            }
        }

        stage('Deploy using Docker Compose') {
            steps {
                sh '''
                    echo "🔥 Stopping old containers to avoid conflicts..."
                    
                    # Navigate to Jenkins workspace where compose file is located
                    cd /var/lib/jenkins/workspace/mean-app-cicd
                    
                    # Stop & remove old containers
                    docker compose down --remove-orphans

                    echo "📥 Pulling latest images from Docker Hub..."
                    docker compose pull

                    echo "🚀 Starting new containers..."
                    docker compose up -d --force-recreate

                    echo "🎉 Deployment completed successfully!"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check the logs."
        }
    }
}
