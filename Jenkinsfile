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
                // Đóng gói ứng dụng thành Docker Image tại máy local
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Quét lỗ hổng Image với timeout 15 phút để tránh bị ngắt giữa chừng
                // Chỉ quét lỗ hổng (vuln) để tăng tốc độ pipeline
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image --timeout 15m --scanners vuln ${DOCKER_IMAGE}"
            }
        }

        stage('5. Deploy to Kubernetes') {
            steps {
                script {
                    // Giải pháp khắc phục lỗi "kubectl not found": Dùng Dockerized kubectl
                    // Dùng mạng host để container này thấy được Minikube đang chạy trên máy Kali
                    withKubeConfig([credentialsId: 'k8s-config']) {
                        sh "docker run --rm --net=host -v /var/jenkins_home/workspace/My-DevSecOps-Project/Kubernetes:/apps bitnami/kubectl:latest apply -f /apps/deployment.yml"
                        sh "docker run --rm --net=host -v /var/jenkins_home/workspace/My-DevSecOps-Project/Kubernetes:/apps bitnami/kubectl:latest apply -f /apps/service.yml"
                    }
                }
            }
        }
    }
}
