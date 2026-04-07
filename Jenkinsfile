pipeline {
    agent any
    tools {
            nodejs "node-18"
        }
    environment {
        IMAGE_NAME = "frontend-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        COMPOSE_FILE = "docker-compose.yml"
    }

    stages {

        stage('Prepare') {
            steps {
                echo "Starting Frontend Deployment - Build ${BUILD_NUMBER}"
            }
        }

        stage('Build App') {
            steps {
                sh '''
                npm ci
                npm run build
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh '''
                echo "Deploying version: $IMAGE_TAG"

                export IMAGE_TAG=$IMAGE_TAG

                docker compose up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "Checking application health..."

                sleep 10
                curl -f http://localhost:3000 || exit 1
                '''
            }
        }

        stage('Cleanup Old Images') {
            steps {
                sh '''
                echo "Cleaning old images (keeping last 3)..."

                docker images $IMAGE_NAME --format "{{.Repository}}:{{.Tag}}" | tail -n +4 | xargs -r docker rmi
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment Successful - Version ${IMAGE_TAG}"
        }

        failure {
            echo "Deployment Failed - Rolling back..."

            sh '''
            PREV_TAG=$(docker images frontend-app --format "{{.Tag}}" | sed -n '2p')

            if [ ! -z "$PREV_TAG" ]; then
                echo "Rolling back to version $PREV_TAG"

                export IMAGE_TAG=$PREV_TAG
                docker compose up -d
            else
                echo " No previous version found"
            fi
            '''
        }
    }
}
