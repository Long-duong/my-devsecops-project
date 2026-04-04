pipeline {
    agent any

    environment {
        // Khai báo biến môi trường để dễ quản lý
        DOCKER_IMAGE = "my-youtube-app:latest"
        DOCKER_HUB_USER = "long12si"
        DOCKER_HUB_REPO = "long12si/my-youtube-app"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Tải code mới nhất từ GitHub
                checkout scm
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    // Quét chất lượng và bảo mật mã nguồn
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                // Đóng gói ứng dụng (Sử dụng --network=host để tránh timeout mạng máy ảo)
                sh "docker build --network=host -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Quét lỗ hổng trong Docker Image trước khi push
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image --timeout 15m --severity HIGH,CRITICAL ${DOCKER_IMAGE}"
            }
        }

        stage('5. Push & K8s Deploy') {
            steps {
                script {
                    // Sử dụng ID 'docker-hub-creds' đã tạo trong Jenkins
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        
                        // Đăng nhập và đẩy lên Docker Hub
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        sh "docker tag ${DOCKER_IMAGE} ${DOCKER_HUB_REPO}:latest"
                        sh "docker push ${DOCKER_HUB_REPO}:latest"
                        
                        // Triển khai thực tế lên cụm k3d
                        sh "kubectl apply -f Kubernetes/deployment.yml"
                        sh "kubectl apply -f Kubernetes/service.yml"
                        
                        // Ép K8s cập nhật bản mới nhất ngay lập tức
                        sh "kubectl rollout restart deployment/youtube-app"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "-- PIPELINE THÀNH CÔNG RỰC RỠ --"
            echo "Ứng dụng đã sẵn sàng tại http://localhost:8081"
        }
        failure {
            echo "-PIPELINE THẤT BẠI- "
            echo "Vui lòng kiểm tra lại logs của từng Stage."
        }
    }
}
