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

        // ✅ SonarQube sur product-service uniquement (test)
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    dir('product-service') {
                        sh '''
                          mvn -B clean verify \
                            org.sonarsource.scanner.maven:sonar-maven-plugin:4.0.0.4121:sonar \
                            -DskipTests=false \
                            -Dsonar.projectKey=product-service
                        '''
                    }
                }
            }
        }

        // ✅ (Optionnel mais recommandé) Bloquer sur le Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
                            ${services.collect{ service -> "docker push ${REGISTRY}/${service}:latest" }.join('\n')}
                        """
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    def images = [
                        "${REGISTRY}/product-service:latest",
                        "${REGISTRY}/order-service:latest",
                        "${REGISTRY}/inventory-service:latest",
                        "${REGISTRY}/notification-service:latest",
                        "${REGISTRY}/api-gateway:latest",
                        "${REGISTRY}/frontend:latest"
                    ]
                    for (img in images) {
                        sh """
                            echo "🔎 Scanning image: ${img}"
                            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image \
                              --severity HIGH,CRITICAL \
                              --exit-code 0 \
                              --no-progress ${img}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                def url = "http://172.189.28.145/"
                currentBuild.description = "<a href='${url}' target='_blank'>🌐 Ouvrir le Frontend</a>"
                echo "✅ Frontend disponible : ${url}"
            }
        }
        failure {
            echo '❌ Pipeline échoué.'
        }
    }
}
