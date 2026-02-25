def dockerImage

pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "ryanline"
        IMAGE_NAME = "simple-webapp"
        IMAGE_TAG = "latest"

        DOCKER_CREDS = "dockerhub-creds"
        KUBECONFIG_CREDENTIAL = "kubeconfig-id"
    }

    stages {
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
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDS) {
                        dockerImage.push(IMAGE_TAG)
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
  steps {
    withCredentials([file(credentialsId: KUBECONFIG_CREDENTIAL, variable: 'KCFG')]) {
      sh '''
        set -e

        # Put kubeconfig somewhere always writable
        KPATCH=/tmp/kubeconfig.patched
        cp "$KCFG" "$KPATCH"
        chmod 600 "$KPATCH"

        # Disable TLS verification for this kubeconfig (quick fix)
        kubectl --kubeconfig="$KPATCH" config set-cluster c-m-k2x2rxxl --insecure-skip-tls-verify=true

        export KUBECONFIG="$KPATCH"

        kubectl get nodes
        kubectl apply -f k8s/deployment.yaml
        kubectl apply -f k8s/service.yaml
        kubectl get pods
        kubectl get svc
      '''
    }
  }
}
    }
}
