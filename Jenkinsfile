pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        IMAGE_NAME = 'afrozrowshan12345'
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
                echo "🔧 Building Docker image..."
                sh "docker build -t $IMAGE_NAME ."
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "🚀 Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: "$DOCKERHUB_CREDENTIALS", 
                                                 passwordVariable: 'DOCKER_PASSWORD', 
                                                 usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    sh "docker push $IMAGE_NAME"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "📦 Deploying to Kubernetes..."
                sh "kubectl apply -f k8s-deployment.yaml"
            }
        }
    }
}


