
pipeline {

    agent any

    environment {
        APP_DIR = '/home/ubuntu/Simple-Calculator-App'
        COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out latest code...'

                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                echo 'Verifying deployment files...'

                sh '''
                    set -e

                    cd ${APP_DIR}

                    echo "Application directory:"
                    pwd

                    echo "Git commit:"
                    git rev-parse --short HEAD

                    echo "Checking required files..."

                    test -f docker-compose.yml
                    test -f nginx.conf
                    test -f .env
                    test -f Dockerfile
                    test -f backend/Dockerfile

                    echo "All required files are present."
                '''
            }
        }

        stage('Docker Compose Config Check') {
            steps {
                echo 'Validating Docker Compose configuration...'

                sh '''
                    set -e

                    cd ${APP_DIR}

                    docker compose -f ${COMPOSE_FILE} config
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'

                sh '''
                    set -e

                    cd ${APP_DIR}

                    docker compose -f ${COMPOSE_FILE} build
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying application...'

                sh '''
                    set -e

                    cd ${APP_DIR}

                    docker compose -f ${COMPOSE_FILE} up -d --build
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                echo 'Checking running containers...'

                sh '''
                    set -e

                    cd ${APP_DIR}

                    sleep 10

                    docker compose -f ${COMPOSE_FILE} ps

                    echo "Docker containers:"
                    docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking application health...'

                sh '''
                    set -e

                    echo "Testing API health endpoint..."

                    for i in 1 2 3 4 5
                    do
                        if curl -fsS http://localhost/api/health
                        then
                            echo
                            echo "API health check passed."
                            exit 0
                        fi

                        echo "API is not ready yet. Retrying in 5 seconds..."
                        sleep 5
                    done

                    echo "API health check failed."

                    exit 1
                '''
            }
        }
    }

    post {

        success {
            echo '''
========================================
DEPLOYMENT SUCCESSFUL
========================================

Simple Calculator App has been deployed.

Application:
http://<EC2-PUBLIC-IP>

API Health:
http://<EC2-PUBLIC-IP>/api/health

========================================
'''
        }

        failure {
            echo '''
========================================
DEPLOYMENT FAILED
========================================

Collecting Docker Compose logs...
========================================
'''

            sh '''
                cd ${APP_DIR} || exit 0

                docker compose -f ${COMPOSE_FILE} ps || true

                docker compose -f ${COMPOSE_FILE} logs --tail=100 || true
            '''
        }

        always {
            echo 'Jenkins pipeline finished.'
        }
    }
}



