pipeline {
    agent any

    tools {
        // Must match the exact name configured in Global Tool Configuration
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
                echo "Installing dependencies..."
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Running unit tests..."
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building local image ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
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
                    // Determine which CD pipeline to trigger based on the active branch
                    def targetPipeline = (env.BRANCH_NAME == 'main') ? 'Deploy_to_main' : 'Deploy_to_dev'
                    echo "Triggering CD pipeline: ${targetPipeline}..."
                    
                    // Trigger the job asynchronously without blocking this pipeline
                    build job: targetPipeline, wait: false
                }
            }
        }
    }

    post {
        always {
            // Securely log out from Docker Hub on completion
            sh 'docker logout'
        }
    }
}
