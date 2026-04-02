pipeline {
    agent any

    environment {
        // Tên Image nội bộ khớp với khai báo trong deployment.yml
        DOCKER_IMAGE = "my-youtube-app:latest"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Lấy code từ GitHub của bạn
                checkout scm
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    // Quét mã nguồn với SonarScanner
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                // Build Image ngay tại máy Kali
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Quét bảo mật Image, giới hạn quét Vulnerability để chạy nhanh hơn
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image --timeout 15m --scanners vuln ${DOCKER_IMAGE}"
            }
        }

        stage('5. Deploy to Kubernetes') {
            steps {
                script {
                    // Sử dụng ${WORKSPACE} để gắn đúng đường dẫn thư mục vào container kubectl
                    withKubeConfig([credentialsId: 'k8s-config']) {
                        sh """
                        docker run --rm --net=host \
                        -v ${WORKSPACE}/Kubernetes:/apps \
                        bitnami/kubectl:latest apply -f /apps/deployment.yml
                        
                        docker run --rm --net=host \
                        -v ${WORKSPACE}/Kubernetes:/apps \
                        bitnami/kubectl:latest apply -f /apps/service.yml
                        """
                    }
                }
            }
        }
    }
}
