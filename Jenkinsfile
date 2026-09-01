pipeline {

    agent {
        label 'agent'
    }

    environment {
        IMAGE_NAME = 'sample-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = 'sample-app-container'
        APP_PORT = '5000'
    }

    stages {

        stage('Verify Agent') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Agent Information"
                    echo "======================================"

                    whoami
                    hostname
                    id
                    docker --version
                    docker ps
                '''
            }
        }

        stage('Clone GitHub Code') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Cloning GitHub repository"
                    echo "======================================"

                    rm -rf application

                    git clone \
                        https://github.com/Nafiya-26/Jenkins-project.git \
                        application

                    cd application

                    git log -1 --oneline

                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Building Docker image"
                    echo "======================================"

                    cd application

                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -t ${IMAGE_NAME}:latest \
                        .

                    echo "Images:"
                    docker images ${IMAGE_NAME}
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Running Docker container"
                    echo "======================================"

                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:5000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    docker ps
                '''
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Validating Application"
                    echo "======================================"

                    sleep 10

                    echo "Container status:"
                    docker ps -a \
                        --filter name=${CONTAINER_NAME}

                    echo "Container logs:"
                    docker logs ${CONTAINER_NAME} --tail 50

                    echo "Checking container..."

                    RUNNING=$(docker inspect \
                        -f '{{.State.Running}}' \
                        ${CONTAINER_NAME})

                    if [ "$RUNNING" != "true" ]; then
                        echo "Container is NOT running"
                        docker logs ${CONTAINER_NAME}
                        exit 1
                    fi

                    echo "Container is running!"

                    echo "Checking application..."

                    curl -f http://localhost:${APP_PORT}

                    echo ""
                    echo "Application validation successful!"
                '''
            }
        }
    }

    post {

        always {
            echo "======================================"
            echo "Cleanup"
            echo "======================================"

            sh '''
                docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                docker rmi ${IMAGE_NAME}:${IMAGE_TAG} 2>/dev/null || true

                docker rmi ${IMAGE_NAME}:latest 2>/dev/null || true

                docker system prune -f || true

                echo "Cleanup completed"
            '''
        }

        success {
            echo "======================================"
            echo "PIPELINE SUCCESS"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "PIPELINE FAILED"
            echo "Check the console output"
            echo "======================================"
        }
    }
}