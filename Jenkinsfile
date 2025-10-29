pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'vennela2012/webapp'
        CONTAINER_PORT = '8084'
    }

    stages {

        stage('Stage 1 - Checkout Code') {
            steps {
                echo 'Cloning source code from GitHub...'
                git branch: 'main', url: 'https://github.com/Vennela020/webapp.git'
            }
        }

        stage('Stage 2 - Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh 'docker build -t ${DOCKER_IMAGE}:latest .'
                }
            }
        }

        stage('Stage 3 - Tag Image') {
            steps {
                echo 'Tagging Docker image...'
                script {
                    sh 'docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}'
                }
            }
        }

        stage('Stage 4 - Push Image to Docker Hub') {
            steps {
                echo 'Pushing image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'githubid', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}'
                    sh 'docker push ${DOCKER_IMAGE}:latest'
                }
            }
        }

        stage('Stage 5 - Remove Local Images') {
            steps {
                echo 'Cleaning up local Docker images...'
                script {
                    sh 'docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} || true'
                    sh 'docker rmi ${DOCKER_IMAGE}:latest || true'
                }
            }
        }

        stage('Stage 6 - Deploy on Server') {
            steps {
                echo 'Deploying application container...'
                script {
                    sh '''
                    docker pull ${DOCKER_IMAGE}:latest
                    docker stop app-container || true
                    docker rm app-container || true
                    docker run -d --name app-container -p ${CONTAINER_PORT}:8080 ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Stage 7 - Verify Deployment') {
            steps {
                echo 'Checking if container is running...'
                script {
                    sh 'docker ps -a'
                    sh 'curl -I http://localhost:${CONTAINER_PORT}'
                }
            }
        }

        stage('Stage 8 - Post Build Cleanup') {
            steps {
                echo 'Post build actions...'
                cleanWs()
            }
        }
    }

    post {
        success {
            echo '✅ Build and deployment successful!'
        }
        failure {
            echo '❌ Build failed. Please check logs.'
        }
    }
}
