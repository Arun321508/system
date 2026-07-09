pipeline {

    agent any

    environment {
        IMAGE_NAME = "springboot-app"
        COMPOSE_FILE = "docker-compose.yml"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/DeepikaCAshok/Employee-Management-Portal.git'
            }
        }


        stage('Build Application') {
            steps {
                sh '''
                chmod +x mvnw || true
                ./mvnw clean package -DskipTests
                '''
            }
        }


        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME} .
                '''
            }
        }


        stage('Deploy') {
            steps {
                sh '''
                echo "Stopping old containers..."

                docker-compose -f ${COMPOSE_FILE} down --remove-orphans || true

                echo "Removing old containers if exist..."

                docker rm -f mysql-container springboot-container || true

                echo "Starting application..."

                docker-compose -f ${COMPOSE_FILE} up -d --build
                '''
            }
        }


        stage('Wait for Application') {
            steps {
                sh '''
                echo "Waiting for Spring Boot..."

                sleep 20

                docker ps
                '''
            }
        }


        stage('Test') {
            steps {
                sh '''
                curl -f http://localhost:8083/employees || exit 1
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
            sh 'docker-compose logs || true'
        }

    }

}
