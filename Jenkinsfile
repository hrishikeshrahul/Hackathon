pipeline {
  agent any

  environment {
    BACKEND_IMAGE = "hrishikeshrahul/mean-backend:latest"
    FRONTEND_IMAGE = "hrishikeshrahul/mean-frontend:latest"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Backend') {
      steps {
        sh '''
          cd backend
          docker build -t backend-temp:latest .
        '''
      }
    }

    stage('Build Frontend') {
      steps {
        sh '''
          cd frontend
          docker build -t frontend-temp:latest .
        '''
      }
    }

    stage('Login & Push Images') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'dockerhub-login',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
          )
        ]) {
          sh '''
            printf "%s" "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

            docker tag backend-temp:latest ${BACKEND_IMAGE}
            docker tag frontend-temp:latest ${FRONTEND_IMAGE}

            docker push ${BACKEND_IMAGE}
            docker push ${FRONTEND_IMAGE}

            docker logout
          '''
        }
      }
    }

    stage('Deploy using Docker Compose') {
      steps {
        sh '''
          docker-compose down || true
          docker-compose up -d --build
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Deployment successful!"
    }
    failure {
      echo "❌ Deployment failed!"
    }
  }
}
