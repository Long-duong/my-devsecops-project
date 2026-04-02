pipeline {
    agent any

    environment {
        // Tên Image nội bộ (không cần Docker Hub)
        DOCKER_IMAGE = "my-youtube-app:latest"
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
                    // Tên 'SonarScanner' và 'SonarServer' phải khớp với cấu hình trong Jenkins Tools/System
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                // Build image ngay tại máy Kali
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Dùng Docker chạy Trivy để quét image vừa build
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_IMAGE}"
            }
        }

        stage('5. Deploy to Kubernetes') {
            steps {
                script {
                    // 'k8s-config' là ID của Secret File bạn đã add trong Credentials
                    withKubeConfig([credentialsId: 'k8s-config']) {
                        sh "kubectl apply -f Kubernetes/deployment.yml"
                        sh "kubectl apply -f Kubernetes/service.yml"
                    }
                }
            }
        }
    }
}
