pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'navaneethakrishna/frontend-app'
        BACKEND_REPO  = 'navaneethakrishna/backend-app'
        IMAGE_TAG     = "${env.GIT_COMMIT.take(7)}"  // short commit hash

        DOCKER_CLIENT_TIMEOUT = '300'
        COMPOSE_HTTP_TIMEOUT  = '300'
    }

    stages {
        // =========================
        // Checkout Code
        // =========================
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/krihnan/Jenkins-Project.git'
            }
        }

        // =========================
        // Frontend Build
        // =========================
        stage('Frontend - Build Docker Image') {
            steps {
                dir('frontend') {
                    sh "docker build -t ${FRONTEND_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        // =========================
        // Backend Build
        // =========================
        stage('Backend - Build Docker Image') {
            steps {
                dir('backend') {
                    sh "docker build -t ${BACKEND_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        // =========================
        // Docker Hub Login
        // =========================
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        // =========================
        // Push Docker Images
        // =========================
        stage('Push Docker Images') {
            steps {
                retry(3) {
                    sh "docker push ${FRONTEND_REPO}:${IMAGE_TAG}"
                    sh "docker push ${BACKEND_REPO}:${IMAGE_TAG}"

                    sh "docker tag ${FRONTEND_REPO}:${IMAGE_TAG} ${FRONTEND_REPO}:latest"
                    sh "docker tag ${BACKEND_REPO}:${IMAGE_TAG} ${BACKEND_REPO}:latest"
                    sh "docker push ${FRONTEND_REPO}:latest"
                    sh "docker push ${BACKEND_REPO}:latest"
                }
            }
        }

        // =========================
        // Deploy Containers
        // =========================
        stage('Deploy Containers') {
            steps {
                script {
                    // Stop old containers
                    sh 'docker rm -f frontend-container || true'
                    sh 'docker rm -f backend-container || true'

                    // Pull latest images
                    sh "docker pull ${FRONTEND_REPO}:latest || true"
                    sh "docker pull ${BACKEND_REPO}:latest || true"

                    // Run backend container
                    def backendRun = sh(
                        script: """
                            docker run -d --restart unless-stopped --name backend-container \
                                -p 5000:8080 \
                                ${BACKEND_REPO}:latest
                        """,
                        returnStatus: true
                    )
                    if (backendRun != 0) {
                        error "Failed to start backend container!"
                    }

                    // Run frontend container
                    def frontendRun = sh(
                        script: """
                            docker run -d --restart unless-stopped --name frontend-container \
                                -p 3000:80 \
                                ${FRONTEND_REPO}:latest
                        """,
                        returnStatus: true
                    )
                    if (frontendRun != 0) {
                        error "Failed to start frontend container!"
                    }
                }
            }
        }
    }
}
