
pipeline {
    agent any

    tools {
        // Asegúrate de que este nombre coincida con el configurado en tus Global Tools
        nodejs 'NodeJS-LTS'
    }

    environment {
        // Variables dinámicas según la rama activa
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        IMAGE_TAG  = "v1.0"
        APP_PORT   = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        CONTAINER_NAME = "${env.BRANCH_NAME == 'main' ? 'app-main' : 'app-dev'}"
    }

    stages {
        stage('Checkout') {
            steps {
                // El pipeline multibranch hace checkout automático de la rama correspondiente
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Compilando la aplicación de NodeJS..."
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Ejecutando pruebas..."
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Construyendo imagen Docker para ${env.BRANCH_NAME}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                echo "Desplegando contenedor en el puerto ${APP_PORT}..."
                script {
                    // 1. Detener y remover el contenedor anterior si existe de forma segura
                    sh """
                        if [ \$(docker ps -a -q -f name=^/${CONTAINER_NAME}\$) ]; then
                            echo "Deteniendo contenedor antiguo para minimizar downtime..."
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                        fi
                    """
                    
                    // 2. Ejecutar el nuevo contenedor (Mapeo dinámico de puertos)
                    // Dentro del contenedor corre en el puerto 3000, expuesto hacia afuera en APP_PORT (3000 o 3001)
                    sh "docker run -d --name ${CONTAINER_NAME} --expose ${APP_PORT} -p ${APP_PORT}:3000 ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }
}