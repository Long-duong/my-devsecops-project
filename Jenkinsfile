pipeline {
    agent any

    environment {
        // Tên Image nội bộ để quét và deploy
        DOCKER_IMAGE = "my-youtube-app:latest"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Tải code từ GitHub của Long-duong
                checkout scm
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    // Quét chất lượng mã nguồn
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
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // QUAN TRỌNG: Quét lỗ hổng bảo mật Image với timeout 15 phút
                // Đã tối ưu chỉ quét lỗ hổng phần mềm (--scanners vuln) để chạy nhanh
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image --timeout 15m --scanners vuln ${DOCKER_IMAGE}"
            }
        }

        stage('5. Deploy to Kubernetes') {
            steps {
                script {
                    // Triển khai lên cụm Minikube (Dùng ID k8s-config đã tạo)
                    withKubeConfig([credentialsId: 'k8s-config']) {
                        sh "kubectl apply -f Kubernetes/deployment.yml"
                        sh "kubectl apply -f Kubernetes/service.yml"
                    }
                }
            }
        }
    }
}
