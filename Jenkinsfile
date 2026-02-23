pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'quangdm006'
        BACKEND_IMAGE   = "${DOCKER_HUB_USER}/backend"
        FRONTEND_IMAGE  = "${DOCKER_HUB_USER}/frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Pulling source code...'
                checkout scm
            }
        }

        stage('Build Images') {
            steps {
                echo 'Building Docker images...'
                sh "docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} -t ${BACKEND_IMAGE}:latest ./backend"
                sh "docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} -t ${FRONTEND_IMAGE}:latest ./frontend"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing images to Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${BACKEND_IMAGE}:latest"
                    sh "docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${FRONTEND_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                echo 'Deploying to Minikube...'
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f k8s/ --validate=false'
                    sh "kubectl set image deployment/backend backend=${BACKEND_IMAGE}:${BUILD_NUMBER}"
                    sh "kubectl set image deployment/frontend frontend=${FRONTEND_IMAGE}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Verify') {
            steps {
                echo 'Verifying deployment...'
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl rollout status deployment/backend --timeout=120s'
                    sh 'kubectl rollout status deployment/frontend --timeout=120s'
                    sh 'kubectl get pods'
                    sh 'kubectl get services'
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline SUCCESS! App deployed to Minikube.'
            echo "Access app at: minikube service nginx --url"
        }
        failure {
            echo '❌ Pipeline FAILED! Check logs above.'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
