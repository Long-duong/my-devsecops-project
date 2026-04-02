pipeline {
    agent any

    environment {
        // Tên Image nội bộ khớp với file deployment.yml
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
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                // Đã sửa lỗi phiên bản bằng cách chỉ định 0.49.1
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image ${DOCKER_IMAGE}"
            }
        }

        stage('5. Deploy to Kubernetes') {
            steps {
                script {
                    // Sử dụng k8s-config để đẩy lên Minikube
                    withKubeConfig([credentialsId: 'k8s-config']) {
                        sh "kubectl apply -f Kubernetes/deployment.yml"
                        sh "kubectl apply -f Kubernetes/service.yml"
                    }
                }
            }
        }
    }
}
