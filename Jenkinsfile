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

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                      echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    '''
                }
            }
        }

        stage('Build & Package Backends') {
            parallel {
                stage('Product Service') {
                    steps {
                        dir('product-service') {
                            sh 'mvn -B clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/product-service:latest ."
                        }
                    }
                }
                stage('Order Service') {
                    steps {
                        dir('order-service') {
                            sh 'mvn -B clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/order-service:latest ."
                        }
                    }
                }
                stage('Inventory Service') {
                    steps {
                        dir('inventory-service') {
                            sh 'mvn -B clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/inventory-service:latest ."
                        }
                    }
                }
                stage('Notification Service') {
                    steps {
                        dir('notification-service') {
                            sh 'mvn -B clean package -DskipTests'
                            sh "docker build -t ${REGISTRY}/notification-service:latest ."
                        }
                    }
                }
                stage('API Gateway') {
                    steps {
                        dir('api-gateway') {
                            sh 'mvn -B clean package -DskipTests'
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

        // ✅ Analyse SonarQube (seulement product-service pour test)
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    dir('product-service') {
                        sh '''
