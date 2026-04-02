pipeline {
    agent any

    environment {
        // Tên Image nội bộ khớp với khai báo trong Kubernetes/deployment.yml
        DOCKER_IMAGE = "my-youtube-app:latest"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Tải code từ GitHub vào Jenkins workspace
                checkout scm
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    // Quét mã nguồn (Đảm bảo container SonarQube đã start trên Kali)
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                // Build file Dockerfile nằm ngay thư mục gốc
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Quét bảo mật image vừa build (phiên bản ổn định 0.49.1)
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image --timeout 15m --scanners vuln ${DOCKER_IMAGE}"
            }
        }

        stage('5. Deploy to Kubernetes') {
            steps {
                script {
                    // Dùng ID 'k8s-config' bạn đã tạo trong Jenkins Credentials
                    // Kỹ thuật dùng dấu gạch ngang '-' để đọc nội dung file trực tiếp, tránh lỗi path
                    withKubeConfig([credentialsId: 'k8s-config']) {
                        sh """
                        cat Kubernetes/deployment.yml | docker run --rm -i --net=host bitnami/kubectl:latest apply -f -
                        cat Kubernetes/service.yml | docker run --rm -i --net=host bitnami/kubectl:latest apply -f -
                        """
                    }
                }
            }
        }
    }
}
