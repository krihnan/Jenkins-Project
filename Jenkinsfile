pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'navaneethakrishna/frontend-app'
        BACKEND_REPO  = 'navaneethakrishna/backend-app'
        IMAGE_TAG     = "${env.GIT_COMMIT.take(7)}"  // short commit hash
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/krihnan/Jenkins-Project.git'
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh "docker build -t ${FRONTEND_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh "docker build -t ${BACKEND_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh """
                docker push ${FRONTEND_REPO}:${IMAGE_TAG}
                docker push ${BACKEND_REPO}:${IMAGE_TAG}

                docker tag ${FRONTEND_REPO}:${IMAGE_TAG} ${FRONTEND_REPO}:latest
                docker tag ${BACKEND_REPO}:${IMAGE_TAG} ${BACKEND_REPO}:latest

                docker push ${FRONTEND_REPO}:latest
                docker push ${BACKEND_REPO}:latest
                """
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                # Remove any existing containers safely
                docker rm -f frontend-container backend-container mysql-container mongo-container || true

                # Remove unused networks
                docker network prune -f

                # Pull latest images
                docker pull ${FRONTEND_REPO}:latest
                docker pull ${BACKEND_REPO}:latest

                # Start everything with docker-compose
                docker compose up -d --remove-orphans
                '''
            }
        }
    }
}
