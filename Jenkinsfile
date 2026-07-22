pipeline {
    agent any

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

        // Ejecuta npm install dentro del agente Docker node:7.8.0
        stage('Build') {
            agent {
                docker {
                    image 'node:7.8.0'
                    reuseNode true
                }
            }
            steps {
                echo "Installing dependencies inside node:7.8.0 agent..."
                sh 'npm install'
            }
        }

        // Ejecuta npm test dentro del agente Docker node:7.8.0
        stage('Test') {
            agent {
                docker {
                    image 'node:7.8.0'
                    reuseNode true
                }
            }
            steps {
                echo "Running unit tests inside node:7.8.0 agent..."
                sh 'npm test'
            }
        }

        // Linting del Dockerfile con Hadolint (corre en el host)
        stage('Lint Dockerfile') {
            steps {
                echo "Linting Dockerfile with Hadolint Container..."
                sh 'docker run --rm -i hadolint/hadolint < Dockerfile'
            }
        }

        // Construcción de la imagen Docker (corre en el host)
        stage('Build Docker Image') {
            steps {
                echo "Building local image ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        // Escaneo de vulnerabilidades con Trivy (corre en el host)
        stage('Security Scan (Trivy)') {
            steps {
                echo "Scanning Docker image for vulnerabilities with Trivy..."
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        // Publicación en Docker Hub (corre en el host)
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

        // Disparar el pipeline de CD correspondiente
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
            sh 'docker logout || true'
        }
    }
}
