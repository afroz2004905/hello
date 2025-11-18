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
    script {
        echo '🐳 Building Docker image...'
        // Ensure the full repository name is included here: afrozrowshan12345/flask-ecommerce
        sh 'docker build -t afrozrowshan12345/flask-ecommerce:13 .' 
    }
}

stage('Push to Docker Hub') {
    script {
        echo '📤 Pushing image to Docker Hub...'
        // CORRECT SYNTAX: Use the required closure structure
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                          usernameVariable: 'DOCKER_USER', 
                                          passwordVariable: 'DOCKER_PASS')]) {
            
            // 1. Log in to Docker using the retrieved variables
            sh "echo \"${DOCKER_PASS}\" | docker login -u ${DOCKER_USER} --password-stdin"
            
            // 2. Push the image
            sh 'docker push afrozrowshan12345/flask-ecommerce:13'
        }
        // IMPORTANT: The login is only valid inside the withCredentials block.
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
                        echo "🌍 Access your app at: http://$(minikube ip):30007"
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
            echo "❌ Deployment failed! Please check Jenkins logs."
        }
    }
}

