pipeline {

    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        IMAGE_NAME = "springboot-app"
        COMPOSE_FILE = "docker-compose.yml"
    }

    stages {

        stage('Checkout') {
            steps {
                cleanWs()

                git branch: 'main',
                    url: 'https://github.com/Arun321508/system.git'
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                mvn clean package -DskipTests
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Stopping old containers..."

                docker compose -f ${COMPOSE_FILE} down --remove-orphans || true

                echo "Removing old containers..."

                docker rm -f springboot-container mysql-container || true

                echo "Building and starting containers..."

                docker compose -f ${COMPOSE_FILE} up -d 
                '''
            }
        }

        stage('Wait for Application') {
            steps {
                sh '''
                echo "Waiting for Spring Boot to start..."

                sleep 30

                docker ps
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                curl --retry 10 --retry-delay 5 --retry-connrefused \
                http://localhost:8083/employees
                '''
            }
        }
    }

    post {

        success {
            echo "Deployment successful"
        }

        failure {
            echo "Deployment failed"

            sh '''
            echo "Running containers:"
            docker ps -a || true

            echo "Docker Compose logs:"
            docker compose -f ${COMPOSE_FILE} logs || true
            '''
        }

        always {
            cleanWs()
        }
    }
}
