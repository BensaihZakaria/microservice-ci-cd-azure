pipeline {
  agent any

  tools {
    maven 'maven3'
    nodejs 'nodejs24'
  }

  environment {
    REGISTRY = "zakaria697"

    // Cache local partagé entre les runs Trivy
    TRIVY_CACHE = "${WORKSPACE}/.trivy-cache"
    // Forcer les registres officiels (évite le fallback instable)
    TRIVY_DB_REPO = "ghcr.io/aquasecurity/trivy-db:2"
    TRIVY_JAVA_DB_REPO = "ghcr.io/aquasecurity/trivy-java-db:1"
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

    // SonarQube (product-service pour l’exemple)
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

    // ---- TRIVY : warm-up + scans robustes ----------------------------------
    stage('Trivy DB Warmup') {
      steps {
        sh """
          mkdir -p ${TRIVY_CACHE}
          # Télécharge la DB vuln et la Java DB une seule fois
          docker run --rm \
            -e TRIVY_DB_REPOSITORY=${TRIVY_DB_REPO} \
            -e TRIVY_JAVA_DB_REPOSITORY=${TRIVY_JAVA_DB_REPO} \
            -v ${TRIVY_CACHE}:/root/.cache/ \
            aquasec/trivy:latest --download-db-only --no-progress --timeout 10m

          docker run --rm \
            -e TRIVY_DB_REPOSITORY=${TRIVY_DB_REPO} \
            -e TRIVY_JAVA_DB_REPOSITORY=${TRIVY_JAVA_DB_REPO} \
            -v ${TRIVY_CACHE}:/root/.cache/ \
            aquasec/trivy:latest --download-java-db-only --no-progress --timeout 10m
        """
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
            retry(2) { // re-tente si souci réseau éphémère
              sh """
                echo "🔎 Scanning image: ${img}"
                docker run --rm \
                  -e TRIVY_DB_REPOSITORY=${TRIVY_DB_REPO} \
                  -e TRIVY_JAVA_DB_REPOSITORY=${TRIVY_JAVA_DB_REPO} \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  -v ${TRIVY_CACHE}:/root/.cache/ \
                  aquasec/trivy:latest image \
                    --timeout 10m \
                    --scanners vuln \
                    --skip-db-update --skip-java-db-update \
                    --severity HIGH,CRITICAL \
                    --exit-code 0 \
                    --no-progress ${img}
              """
            }
          }
        }
      }
    }
    // ------------------------------------------------------------------------
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
