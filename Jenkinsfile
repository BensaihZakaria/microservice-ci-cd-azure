pipeline {
    agent any

    tools {
        maven 'maven3'
        nodejs 'nodejs24'
    }

    environment {
        REGISTRY = "zakaria697"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BensaihZakaria/microservice-ci-cd-azure.git'
            }
        }

        stage('Build & Package Backends') {
            parallel {
                stage('Product Service') {
                    steps {
                        dir('product-service') {
                            sh 'mvn clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/product-service:latest ."
                        }
                    }
                }
                stage('Order Service') {
                    steps {
                        dir('order-service') {
                            sh 'mvn clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/order-service:latest ."
                        }
                    }
                }
                stage('Inventory Service') {
                    steps {
                        dir('inventory-service') {
                            sh 'mvn clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/inventory-service:latest ."
                        }
                    }
                }
                stage('Notification Service') {
                    steps {
                        dir('notification-service') {
                            sh 'mvn clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/notification-service:latest ."
                        }
                    }
                }
                stage('API Gateway') {
                    steps {
                        dir('api-gateway') {
                            sh 'mvn clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/api-gateway:latest ."
                        }
                    }
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build --prod'
                    sh "docker build -t ${REGISTRY}/frontend:latest ."
                }
            }
        }

        stage('Push Docker Images to DockerHub') {
            steps {
                script {
                    def services = [
                        'product-service',
                        'order-service',
                        'inventory-service',
                        'notification-service',
                        'api-gateway',
                        'frontend'
                    ]
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            ${services.collect{ service -> """
                            docker push ${REGISTRY}/${service}:latest
                            """ }.join('\n')}
                        """
                    }
                }
            }
        }
    }
}
