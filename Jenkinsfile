
pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/Game"
        IMAGE_NAME = "indie-gems"
        IMAGE_TAG  = "latest"
        CONTAINER_NAME = "indie-gems-container"
        DOCKER_HUB_USER = "sukesh632k"
        PORT = "4000"   // External port for app
    }

    stages {
        stage('Checkout Code') {
            steps {
                dir("${WORK_DIR}") {
                    git branch: 'main', url: 'https://github.com/SukeshKaicharla/Games.git'
                }
            }
        }
        stage('mvn')
        {
            sh 'mvn clean package'
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
                    credentialsId: 'docker_cred',  // Jenkins credentials ID
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}

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

        stage('Docker Swarm Deploy') {
            steps {
                sh '''
                    docker service update --image ${IMAGE_NAME}:${IMAGE_TAG} gameserv || \
                    docker service create --name gameserv -p 8009:8080 --replicas 10 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished and updated! Check http://13.204.85.107:3000"

        }
    }
}

