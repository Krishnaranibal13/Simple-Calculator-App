
pipeline {

    agent any

    environment {
        COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {

        stage('Verify Files') {
            steps {
                echo 'Verifying deployment files...'

                sh '''
                    set -e

                    echo "Jenkins workspace:"
                    pwd

                    echo "Workspace contents:"
                    ls -la

                    echo "Checking required files..."

                    test -f docker-compose.yml
                    test -f nginx.conf
                    test -f Dockerfile
                    test -f backend/Dockerfile

                    echo "All required files are present."
                '''
            }
        }

        stage('Prepare Environment') {
            steps {
                echo 'Preparing environment file...'

                sh '''
                    set -e

                    if [ ! -f .env ]; then
                        echo "ERROR: .env file is missing from Jenkins workspace."
                        exit 1
                    fi

                    echo ".env file found."
                '''
            }
        }

        stage('Docker Compose Config Check') {
            steps {
                echo 'Validating Docker Compose configuration...'

                sh '''
                    set -e

                    docker compose -f ${COMPOSE_FILE} config
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'

                sh '''
                    set -e

                    docker compose -f ${COMPOSE_FILE} build
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying application...'

                sh '''
                    set -e

                    docker compose -f ${COMPOSE_FILE} up -d
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                echo 'Checking running containers...'

                sh '''
                    set -e

                    sleep 10

                    docker compose -f ${COMPOSE_FILE} ps

                    echo "Docker containers:"
                    docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking API health...'

                sh '''
                    set -e

                    echo "Testing http://localhost/api/health"

                    for i in 1 2 3 4 5
                    do
                        if curl -fsS http://localhost/api/health
                        then
                            echo
                            echo "API health check passed."
                            exit 0
                        fi

                        echo "API is not ready. Retrying in 5 seconds..."
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

Simple Calculator App deployed successfully.

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

Docker Compose status:
'''

            sh '''
                docker compose -f ${COMPOSE_FILE} ps || true

                echo "Recent container logs:"

                docker compose -f ${COMPOSE_FILE} logs --tail=100 || true
            '''
        }

        always {
            echo 'Jenkins pipeline finished.'
        }
    }
}





