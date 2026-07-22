pipeline {
    agent any

    tools {
        // Herramienta configurada en Global Tools con Node.js 7.8.0
        nodejs 'node'
    }

    environment {
        // Usuario de Docker Hub
        DOCKER_HUB_USER = 'juandago'
        
        // Variables dinámicas según la rama activa
        IMAGE_NAME     = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG      = 'v1.0'
        APP_PORT       = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        CONTAINER_NAME = "${env.BRANCH_NAME == 'main' ? 'app-main' : 'app-dev'}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Compilando la aplicación con Node.js..."
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Ejecutando pruebas unitarias..."
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Construyendo la imagen local ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Publicando imagen en Docker Hub..."
                // Usa las credenciales 'dockerhub-creds' para autenticarse de forma segura
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    
                    // Etiquetar la imagen con el formato de tu repositorio personal: juandago/nodemain:v1.0 o juandago/nodedev:v1.0
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    
                    // Subir la imagen a Docker Hub
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Desplegando contenedor ${CONTAINER_NAME} en el puerto ${APP_PORT}..."
                script {
                    // Limpieza selectiva: solo elimina el contenedor del entorno activo sin afectar al otro
                    sh """
                        if [ \$(docker ps -a -q -f name=^/${CONTAINER_NAME}\$) ]; then
                            echo "Deteniendo contenedor anterior..."
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                        fi
                    """
                    
                    // Ejecuta la imagen recién subida
                    sh "docker run -d --name ${CONTAINER_NAME} --expose ${APP_PORT} -p ${APP_PORT}:3000 ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }
    
    post {
        always {
            // Limpieza de sesión para no dejar abierta la autenticación de Docker en el servidor
            sh 'docker logout'
        }
    }
}
