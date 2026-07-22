pipeline {
    // Run the pipeline inside an isolated Node.js 7.8.0 Docker container
    agent {
        docker {
            image 'node:7.8.0'
            // Mount the host Docker socket to enable Docker-in-Docker operations
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
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
                echo "Installing dependencies inside node:7.8.0 agent..."
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Running unit tests inside node:7.8.0 agent..."
                sh 'npm test'
            }
        }

        stage('Lint Dockerfile') {
            steps {
                echo "Linting Dockerfile with Hadolint..."
                sh 'hadolint Dockerfile'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building local image ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                echo "Scanning Docker image for vulnerabilities with Trivy..."
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
