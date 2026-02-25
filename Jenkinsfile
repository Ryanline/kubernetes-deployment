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
                        KPATCH=/tmp/kubeconfig.patched
                        cp "$KCFG" "$KPATCH"
                        chmod 600 "$KPATCH"

                        # DEBUG
                        ls -la "$KCFG" "$KPATCH" /tmp

                        # Force TLS skip on every command (works even if kubeconfig doesn't update cleanly)
                        kubectl --kubeconfig="$KPATCH" --insecure-skip-tls-verify=true get nodes
                        kubectl --kubeconfig="$KPATCH" --insecure-skip-tls-verify=true apply -f k8s/deployment.yaml
                        kubectl --kubeconfig="$KPATCH" --insecure-skip-tls-verify=true apply -f k8s/service.yaml
                        kubectl --kubeconfig="$KPATCH" --insecure-skip-tls-verify=true get pods
                        kubectl --kubeconfig="$KPATCH" --insecure-skip-tls-verify=true get svc
                    '''
                }
            }
        }
    }
}
