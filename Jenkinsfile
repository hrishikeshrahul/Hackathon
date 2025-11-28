pipeline {
  agent any

  environment {
    BACKEND_IMAGE = "hrishikeshrahul/mean-backend:latest"
    FRONTEND_IMAGE = "hrishikeshrahul/mean-frontend:latest"
    TEMP_BACKEND = "backend-temp:latest"
    TEMP_FRONTEND = "frontend-temp:latest"
  }

  stages {
    stage('Checkout') {
      steps {
        // uses the Jenkinsfile from SCM by default when job is configured to pull from repo
        checkout scm
      }
    }

    stage('Build Backend') {
      steps {
        sh '''
          set -euo pipefail
          cd backend
          docker build -t ${TEMP_BACKEND} .
        '''
      }
    }

    stage('Build Frontend') {
      steps {
        sh '''
          set -euo pipefail
          cd frontend
          docker build -t ${TEMP_FRONTEND} .
        '''
      }
    }

    stage('Login & Push Images') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-login',
                                         usernameVariable: 'DOCKER_USER',
                                         passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            set -euo pipefail
            # pipe the token into docker login (must use token starting with dckr_pat_... stored in Jenkins)
            printf '%s' "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

            # retag to your Docker Hub namespace
            docker tag ${TEMP_BACKEND} ${BACKEND_IMAGE} || true
            docker tag ${TEMP_FRONTEND} ${FRONTEND_IMAGE} || true

            # push
            docker push ${BACKEND_IMAGE}
            docker push ${FRONTEND_IMAGE}

            # logout to avoid leaving credentials on node
            docker logout
          '''
        }
      }
    }

    stage('Deploy using Docker Compose') {
      steps {
        sh '''
          set -euo pipefail
          # make sure docker-compose file is present at repo root
          docker compose down || true
          docker compose up -d --build
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline finished successfully."
    }
    failure {
      echo "❌ Pipeline failed. Check logs."
    }
  }
}
