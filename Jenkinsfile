pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = 'docker.io/ashsajal/myapp'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ashsajal/myapp.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    sed -i 's|image:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|' k8s/deployment.yaml
                    kubectl apply -f k8s/
                    kubectl rollout status deployment/myapp-deployment
                    """
                }
            }
        }
    }

    post {
        failure {
            echo 'Something went wrong. Attempting rollback.'
            sh 'kubectl rollout undo deployment/myapp-deployment || echo "No rollback available."'
        }
    }
}
