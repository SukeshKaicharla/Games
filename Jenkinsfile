pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/Game"
        IMAGE_NAME = "indie-gems"
        IMAGE_TAG  = "latest"
        CONTAINER_NAME = "indie-gems-container"
        DOCKER_HUB_USER = "sukesh632k"
        PORT = "3000"   // External port for app
    }

    stages {
        stage('Checkout Code') {
            steps {
                dir("${WORK_DIR}") {
                    git branch: 'main', url: 'https://github.com/SukeshKaicharla/Indie_Gems_Portal.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${WORK_DIR}/Dockerfile ${WORK_DIR}
                    '''
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',  // Jenkins credentials ID
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "⚡ Logging into DockerHub..."
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        echo "⚡ Tagging image..."
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}

                        echo "⚡ Pushing image..."
                        docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        docker rm -f ${CONTAINER_NAME} || true
                        docker pull ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker run -d --name ${CONTAINER_NAME} -p ${PORT}:80 ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline finished! Check http://13.204.85.107:3000"
        }
    }
}
