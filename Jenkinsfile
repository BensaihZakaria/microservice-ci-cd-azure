pipeline {
    agent any

    tools {
        maven 'maven3'
        nodejs 'nodejs24'
    }

    environment {
        REGISTRY = "zakaria697" // ton nom DockerHub
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
        stage('SonarQube Analysis') {
             environment {
                  SONAR_TOKEN = credentials('sonarqube-token')
         }
          steps {
                 withSonarQubeEnv('SonarQube') {
                     sh 'mvn sonar:sonar -Dsonar.projectKey=microservice-ci-cd-azure -Dsonar.host.url=http://localhost:9000 -Dsonar.login=$SONAR_TOKEN'
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
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin
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
