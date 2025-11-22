pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        IMAGE_NAME = 'afrozrowshan12345/flask-ecommerce'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "✅ Checking out source code from GitHub (main branch)..."
                git branch: 'main', url: 'https://github.com/afroz2004905/hello.git'
                sh 'ls -l'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🐳 Building Docker image..."
                    sh '''
                        docker build -t $IMAGE_NAME:$BUILD_NUMBER .
                        docker images | grep flask-ecommerce
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "📤 Pushing image to Docker Hub..."
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $IMAGE_NAME:$BUILD_NUMBER
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    echo "⚙️ Deploying to Minikube..."
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        echo "🔧 Updating image version in deployment file..."
                        sed -i "s|image: .*|image: $IMAGE_NAME:$BUILD_NUMBER|" deployment.yaml

                        echo "🚀 Applying Kubernetes deployment..."
                        kubectl apply -f deployment.yaml --validate=false --insecure-skip-tls-verify

                        echo "⏳ Waiting for rollout to complete..."
                        kubectl rollout status deployment/flask-app --timeout=90s

                        echo "🎉 Deployment successful!"
                        echo "🌍 App running at: http://$(minikube ip):30007"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ All stages completed successfully!"
        }
        failure {
            echo "❌ Deployment failed! Check Jenkins logs."
        }
    }
}

