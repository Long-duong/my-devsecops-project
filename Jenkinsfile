pipeline {
    agent any

    environment {
        // Cấu hình tên Image và Docker Hub
        DOCKER_IMAGE = "my-youtube-app:latest"
        DOCKER_HUB_USER = "long12si"
        DOCKER_HUB_REPO = "long12si/my-youtube-app"
        // Đường dẫn file cấu hình K8s đã được cấp quyền cho Jenkins
        KUBE_CONFIG = "--kubeconfig=/var/lib/jenkins/.kube/config"
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
                    // Quét chất lượng và bảo mật mã nguồn (SAST)
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
                // Quét lỗ hổng bảo mật Image (SCA) trước khi đẩy lên Hub
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image --timeout 15m --severity HIGH,CRITICAL ${DOCKER_IMAGE}"
            }
        }

        stage('5. Push & K8s Deploy') {
            steps {
                script {
                    // Sử dụng ID 'docker-hub-creds' để Login Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        
                        // Bước A: Push Image lên Docker Hub
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        sh "docker tag ${DOCKER_IMAGE} ${DOCKER_HUB_REPO}:latest"
                        sh "docker push ${DOCKER_HUB_REPO}:latest"
                        
                        // Bước B: Triển khai lên cụm k3d (Sử dụng flag chỉ định file config)
                        sh "kubectl apply -f Kubernetes/deployment.yml ${KUBE_CONFIG}"
                        sh "kubectl apply -f Kubernetes/service.yml ${KUBE_CONFIG}"
                        
                        // Bước C: Restart để đảm bảo Pod chạy bản cập nhật mới nhất
                        sh "kubectl rollout restart deployment/youtube-app ${KUBE_CONFIG}"
                    }
                }
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
