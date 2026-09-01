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

        stage('Install Required Tools') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Installing required tools"
                    echo "======================================"

                    sudo apt-get update

                    # Install Git
                    sudo apt-get install -y git

                    # Install Docker if not installed
                    if ! command -v docker >/dev/null 2>&1; then
                        echo "Docker is not installed. Installing..."
                        sudo apt-get install -y docker.io
                    else
                        echo "Docker is already installed"
                    fi

                    echo "Git version:"
                    git --version

                    echo "Docker version:"
                    docker --version
                '''
            }
        }

        stage('Configure Docker Access') {
            steps {
                sh '''
                    echo "====================================="
                    echo "Configuring Docker access"
                    echo "====================================="

                    # Add Jenkins user to Docker group
                    sudo usermod -aG docker jenkins || true
                    sudo usermod -aG docker ubuntu || true

                    # Check Docker service
                    sudo systemctl start docker
                    sudo systemctl enable docker

                    echo "Docker service status:"
                    sudo systemctl is-active docker
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

                    git clone https://github.com/Nafiya-26/Jenkins-project.git application

                    cd application

                    echo "Repository cloned successfully"

                    echo "Latest commit:"
                    git log -1 --oneline

                    echo "Application files:"
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

                    echo "Docker image built successfully"

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

                    # Remove existing container if present
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:5000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    echo "Container started"

                    docker ps
                '''
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Validating application"
                    echo "======================================"

                    echo "Waiting for application to start..."
                    sleep 10

                    echo ""
                    echo "Container status:"
                    docker ps -a --filter name=${CONTAINER_NAME}

                    echo ""
                    echo "Container logs:"
                    docker logs ${CONTAINER_NAME} --tail 50

                    echo ""
                    echo "Checking container status..."

                    RUNNING=$(docker inspect \
                        -f '{{.State.Running}}' \
                        ${CONTAINER_NAME})

                    if [ "$RUNNING" != "true" ]; then
                        echo "ERROR: Container is not running"
                        docker logs ${CONTAINER_NAME}
                        exit 1
                    fi

                    echo "Container is running successfully"

                    echo ""
                    echo "Checking application on port 5000..."

                    curl -f http://localhost:5000

                    echo ""
                    echo "Application validation successful!"
                '''
            }
        }
    }

    post {

        always {
            echo "======================================"
            echo "Post processing / Cleanup"
            echo "======================================"

            sh '''
                echo "Stopping and removing container..."

                docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                echo "Removing Docker images..."

                docker rmi ${IMAGE_NAME}:${IMAGE_TAG} 2>/dev/null || true
                docker rmi ${IMAGE_NAME}:latest 2>/dev/null || true

                echo "Cleaning unused Docker resources..."

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
            echo "Check the Jenkins console output"
            echo "======================================"
        }
    }
}