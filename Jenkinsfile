pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "Ryanline"
        IMAGE_NAME = "simple-webapp"
        IMAGE_TAG = "latest"
        DOCKER_CREDS = "docker-pass"
        KUBECONFIG_CREDENTIAL = "kubeconfig-id"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'git@github.com:Ryanline/kubernetes-deployment.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIAL, variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }
    }
}
