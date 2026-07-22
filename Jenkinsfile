pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        DOCKER_HUB_USER = 'juandago'
        IMAGE_NAME     = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG      = 'v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        // --- TASK: Hadolint check before build ---
        stage('Lint Dockerfile') {
            steps {
                echo "Linting Dockerfile with Hadolint..."
                // Runs Hadolint against your Dockerfile
                sh 'hadolint Dockerfile'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building local image ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        // --- TASK: Vulnerability scan using Trivy ---
        stage('Security Scan (Trivy)') {
            steps {
                echo "Scanning Docker image for vulnerabilities with Trivy..."
                // Scans the local docker image for HIGH and CRITICAL vulnerabilities
                sh "trivy image --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing image to Docker Hub (${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG})..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Trigger CD Pipeline') {
            steps {
                script {
                    def targetPipeline = (env.BRANCH_NAME == 'main') ? 'Deploy_to_main' : 'Deploy_to_dev'
                    echo "Triggering CD pipeline: ${targetPipeline}..."
                    build job: targetPipeline, wait: false
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
