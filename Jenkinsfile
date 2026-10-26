pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        PROJECT_DIR = '/opt/employee-directory/employee-directory-search-system'
        COMPOSE_PROJECT_NAME = 'employee-directory-search-system'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log -1 --oneline'
            }
        }

        stage('Verify Docker') {
            steps {
                sh '''
                    docker --version
                    docker compose version
                    docker ps
                '''
            }
        }

        stage('Validate Configuration') {
            steps {
                sh '''
                    test -f docker-compose.yml
                    test -f backend/Dockerfile
                    test -f frontend/Dockerfile
                    test -f "$PROJECT_DIR/.env"
                    docker compose \
                      -p "$COMPOSE_PROJECT_NAME" \
                      --env-file "$PROJECT_DIR/.env" \
                      config --quiet
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker compose \
                      -p "$COMPOSE_PROJECT_NAME" \
                      --env-file "$PROJECT_DIR/.env" \
                      build
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker compose \
                      -p "$COMPOSE_PROJECT_NAME" \
                      --env-file "$PROJECT_DIR/.env" \
                      up -d
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                sh '''
                    docker compose \
                      -p "$COMPOSE_PROJECT_NAME" \
                      --env-file "$PROJECT_DIR/.env" \
                      ps

                    docker inspect \
                      --format='{{.State.Status}}' \
                      employee_directory_backend

                    docker inspect \
                      --format='{{.State.Status}}' \
                      employee_directory_frontend

                    docker inspect \
                      --format='{{.State.Health.Status}}' \
                      employee_directory_db
                '''
            }
        }

        stage('Application Test') {
            steps {
                sh '''
                    set -eu

                    curl --fail --silent --show-error \
                      --retry 5 --retry-delay 3 --retry-connrefused \
                      http://localhost/

                    curl --fail --silent --show-error \
                      --retry 5 --retry-delay 3 --retry-connrefused \
                      http://localhost/api/employees/all

                    echo "Application tests passed."
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Employee Directory deployment and tests completed.'
        }
        failure {
            echo 'FAILURE: Check the stage logs above to identify the issue.'
        }
        always {
            echo 'Jenkins pipeline finished.'
        }
    }
}
