pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        IMAGE_NAME = 'afrozrowshan12345/flask-ecommerce'
        IMAGE_TAG = '13'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "✅ Checking out source code from GitHub (main branch)..."
                git branch: 'main', url: 'https://github.com/afroz2004905/hello.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '📤 Pushing image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}",
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    echo '⚙️ Deploying to Minikube...'

                    sh "sed -i 's|/root/docker-project/minikube_data/.minikube|/var/lib/jenkins/.minikube|g' ${KUBECONFIG}"

                    sh """sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml"""

                    echo '🚀 Applying Kubernetes deployment...'
                    sh 'kubectl apply -f deployment.yaml --validate=false --insecure-skip-tls-verify'

                    echo '⏳ Waiting for rollout to complete...'
                    sh 'kubectl rollout status deployment/flask-app'
                }
            }
        }

    }   // ✅ THIS WAS MISSING — closes stages {}

    post {
        success {
            echo "✅ Pipeline Success! Application deployed and rolled out."
        }
        failure {
            echo "❌ Pipeline FAILED! Check console output for deployment errors."
        }
    }
}

