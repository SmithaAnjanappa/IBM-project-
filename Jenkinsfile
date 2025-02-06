pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'your-docker-registry-url'
        DOCKER_CREDENTIALS = 'docker-credentials-id'
        KUBECONFIG_CREDENTIALS = 'kubeconfig-credentials-id'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_REGISTRY}/frontend:latest -f Dockerfile-frontend .'
                    sh 'docker build -t ${DOCKER_REGISTRY}/backend:latest -f Dockerfile-backend .'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'docker run --rm ${DOCKER_REGISTRY}/backend:latest pytest tests/' // Example test step
                }
            }
        }

        stage('Push Images') {
            steps {
                withDockerRegistry([credentialsId: DOCKER_CREDENTIALS, url: "https://${DOCKER_REGISTRY}"]) {
                    sh 'docker push ${DOCKER_REGISTRY}/frontend:latest'
                    sh 'docker push ${DOCKER_REGISTRY}/backend:latest'
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS, variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f kubernetes/deployment.yaml'
                    sh 'kubectl apply -f kubernetes/service.yaml'
                    sh 'kubectl apply -f kubernetes/ingress.yaml'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/logs/*.log', allowEmptyArchive: true
            cleanWs()
        }
    }
}
