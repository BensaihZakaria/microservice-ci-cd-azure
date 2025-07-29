pipeline {
    agent any

    tools {
        maven 'maven3'
        nodejs 'nodejs24'
    }

    environment {
        REGISTRY = "monacrtest.azurecr.io"
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
                            sh 'docker build -t product-service:latest .'
                        }
                    }
                }
                stage('Order Service') {
                    steps {
                        dir('order-service') {
                            sh 'mvn clean package -DskipTests'
                            sh 'docker build -t order-service:latest .'
                        }
                    }
                }
                stage('Inventory Service') {
                    steps {
                        dir('inventory-service') {
                            sh 'mvn clean package -DskipTests'
                            sh 'docker build -t inventory-service:latest .'
                        }
                    }
                }
                stage('Notification Service') {
                    steps {
                        dir('notification-service') {
                            sh 'mvn clean package -DskipTests'
                            sh 'docker build -t notification-service:latest .'
                        }
                    }
                }
                stage('api-gateway') {
                    steps {
                        dir('api-gateway') {
                            sh 'mvn clean package -DskipTests'
                            sh 'docker build -t api-gateway:latest .'
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
                    sh 'docker build -t frontend:latest .'
                }
            }
        }

        stage('Push Docker Images to ACR') {
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
                    withCredentials([usernamePassword(credentialsId: 'acr-credentials', usernameVariable: 'ACR_USERNAME', passwordVariable: 'ACR_PASSWORD')]) {
                        sh """
                            echo \$ACR_PASSWORD | docker login ${REGISTRY} -u \$ACR_USERNAME --password-stdin
                            ${services.collect{ service -> """
                            docker tag ${service}:latest ${REGISTRY}/${service}:latest
                            docker push ${REGISTRY}/${service}:latest
                            """ }.join('\n')}
                        """
                    }
                }
            }
        }

        // D'autres stages plus tard (SonarQube, Trivy, AKS, etc)
    }
}
