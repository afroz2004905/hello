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
                echo '🐳 Building Docker image...'
                sh 'docker build -t $IMAGE_NAME:13 .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '📤 Pushing image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $IMAGE_NAME:13'
                }
            }
        }
 stage('Deploy to Minikube') {
    steps {
        script {
            echo '⚙️ Deploying to Minikube...'
            // 1. ADD THIS LINE: Fixes the internal certificate paths in the kubeconfig file
            sh "sed -i 's|/root/docker-project/minikube_data/.minikube|/var/lib/jenkins/.minikube|g' /var/lib/jenkins/.kube/config"
            // 2. FIX IMAGE TAG
            sh 'sed -i s|image: .*|image: afrozrowshan12345/flask-ecommerce:13| deployment.yaml' 
            sh 'echo 🚀 Applying Kubernetes deployment...'
            sh 'kubectl apply -f deployment.yaml --validate=false --insecure-skip-tls-verify'
            
            // Wait for rollout to ensure deployment is ready
            sh 'kubectl rollout status deployment/flask-app' 
        }
    }
} // <--- THIS BRACE CLOSES THE 'Deploy to Minikube' STAGE (Line 57)

} // <--- **YOU MUST ADD THIS CLOSING BRACE TO END THE 'stages' BLOCK** (Line 58 should be here)

    post { // <--- NOW THE 'post' BLOCK IS IN THE CORRECT LOCATION
        success {
            echo "✅ All stages completed successfully!"
        }
        failure {
            echo "❌ Deployment failed! Please check Jenkins logs."
        }
    }
} // <--- THIS CLOSES THE 'pipeline' BLOCK

// The final '}' in your provided script seems extraneous and should be removed.
