pipeline {
    agent any

    environment {
        REGISTRY = "tonregistry.azurecr.io" // Ou ton Docker Hub, à personnaliser plus tard
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone le repo actuel
                git 'https://github.com/BensaihZakaria/microservice-ci-cd-azure.git'
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
                stage('Gateway Service') {
                    steps {
                        dir('gateway-service') {
                            sh 'mvn clean package -DskipTests'
                            sh 'docker build -t gateway-service:latest .'
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
        // On ajoutera ici plus tard :
        // - Push Docker Registry (Azure CR ou Docker Hub)
        // - SonarQube, Trivy, etc.
        // - Déploiement Azure AKS
    }
}
