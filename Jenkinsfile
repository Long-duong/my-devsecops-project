pipeline {
    agent any

    environment {
        // Tên Image nội bộ khớp với file deployment.yml
        DOCKER_IMAGE = "my-youtube-app:latest"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Tải code từ GitHub
                checkout scm
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    // Đảm bảo SonarQube Server đang chạy trên Kali
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                // Build Image ngay tại máy local
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Quét lỗ hổng Image (Dùng bản 0.49.1 ổn định)
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image --timeout 15m --scanners vuln ${DOCKER_IMAGE}"
            }
        }

        stage('5. Kubernetes Deploy') {
            steps {
                script {
                    // Sử dụng k8s-config để kết nối Minikube
                    // Thêm --validate=false để tránh lỗi timeout khi kubectl cố kết nối lấy schema từ server
                    withKubeConfig([credentialsId: 'k8s-config']) {
                        sh """
                        kubectl apply -f Kubernetes/deployment.yml --validate=false
                        kubectl apply -f Kubernetes/service.yml --validate=false
                        """
                    }
                }
            }
        }
    }
}
