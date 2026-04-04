pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "my-youtube-app:latest"
        DOCKER_HUB_REPO = "long12si/my-youtube-app"
        // Đường dẫn file config ĐÃ COPY VÀO TRONG CONTAINER
        KUBE_CONFIG_PATH = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                sh "docker build --network=host -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Đổi thành nháy kép để nhận biến DOCKER_IMAGE
                sh """
                docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy:0.49.1 image \
                --timeout 15m \
                --severity HIGH,CRITICAL \
                --exit-code 1 \
                ${DOCKER_IMAGE}
                """
            }
        }

        stage('5. Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker tag ${DOCKER_IMAGE} ${DOCKER_HUB_REPO}:latest
                    docker push ${DOCKER_HUB_REPO}:latest
                    """
                }
            }
        }

        stage('6. Deploy to k3d (Kubernetes)') {
            steps {
                // Không cần withCredentials file nếu đã dùng docker cp config vào container
                sh """
                kubectl apply -f Kubernetes/deployment.yml --kubeconfig=${KUBE_CONFIG_PATH}
                kubectl apply -f Kubernetes/service.yml --kubeconfig=${KUBE_CONFIG_PATH}
                kubectl rollout restart deployment/youtube-app --kubeconfig=${KUBE_CONFIG_PATH}
                """
            }
        }
    }

    post {
        success {
            echo "--      PIPELINE THÀNH CÔNG RỰC RỠ              --"
            echo "-- Ứng dụng đã được triển khai lên Kubernetes   --"
        }
        failure {
            echo "--           PIPELINE THẤT BẠI                  --"
            echo "-- Vui lòng kiểm tra Console Log để sửa lỗi     --"
        }
    }
}
