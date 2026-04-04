pipeline {
    agent any

    environment {
        // Cấu hình tên Image và Docker Hub
        DOCKER_IMAGE = "my-youtube-app:latest"
        DOCKER_HUB_REPO = "long12si/my-youtube-app"
        // Đường dẫn file cấu hình Kubernetes bên trong Container Jenkins
        KUBE_CONFIG_PATH = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Tải mã nguồn mới nhất từ GitHub
                checkout scm
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    // Quét chất lượng mã nguồn (SAST)
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                // Đóng gói ứng dụng thành Docker Image
                sh "docker build --network=host -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Quét lỗ hổng bảo mật Image (SCA)
                // Đặt --exit-code 0 để Pipeline không bị dừng khi phát hiện lỗ hổng
                sh """
                docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy:0.49.1 image \
                --timeout 15m \
                --severity HIGH,CRITICAL \
                --exit-code 0 \
                ${DOCKER_IMAGE}
                """
            }
        }

        stage('5. Push Image') {
            steps {
                // Sử dụng Credentials 'docker-hub-creds' đã cấu hình trong Jenkins
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
                // Triển khai ứng dụng lên cụm k3d sử dụng file config đã được map vào container
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
            echo "--------------------------------------------------"
            echo "--      PIPELINE THÀNH CÔNG RỰC RỠ              --"
            echo "-- Ứng dụng đã được triển khai lên Kubernetes   --"
            echo "--------------------------------------------------"
        }
        failure {
            echo "--------------------------------------------------"
            echo "--           PIPELINE THẤT BẠI                  --"
            echo "-- Vui lòng kiểm tra Console Log để sửa lỗi     --"
            echo "--------------------------------------------------"
        }
    }
}
