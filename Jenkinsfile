pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "my-youtube-app:latest"
    }
    stages {
        stage('1. Code Checkout') { steps { checkout scm } }
        
        stage('2. SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarServer') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=youtube-app -Dsonar.sources=."
                    }
                }
            }
        }
        
        stage('3. Docker Build') { steps { sh "docker build -t ${DOCKER_IMAGE} ." } }
        
        stage('4. Trivy Security Scan') {
            steps {
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.49.1 image --timeout 15m --scanners vuln ${DOCKER_IMAGE}"
            }
        }
        
        stage('5. Kubernetes Deploy') {
            steps {
                withKubeConfig([credentialsId: 'k8s-config']) {
                    // Bây giờ đã có thể gõ kubectl trực tiếp vì ta vừa cài ở Bước 2
                    sh "kubectl apply -f Kubernetes/deployment.yml"
                    sh "kubectl apply -f Kubernetes/service.yml"
                }
            }
        }
    }
}
