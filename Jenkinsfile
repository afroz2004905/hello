pipeline {
    // Defines the execution agent
    agent any

    // Define environment variables for clean access
    environment {
        // Credential ID for Docker Hub (Ensure this matches your Jenkins credential ID)
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        // Full Image Name to be built and pushed
        IMAGE_NAME = 'afrozrowshan12345/flask-ecommerce'
        // Consistent tag for the build/deploy
        IMAGE_TAG = '13'
        // Kubeconfig path accessible by the Jenkins user
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    // Define the stages of the pipeline
    stages {
        
        stage('Checkout') {
            steps {
                echo "✅ Checking out source code from GitHub (main branch)..."
                // This step ensures the workspace is clean and ready
                git branch: 'main', url: 'https://github.com/afroz2004905/hello.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                // Builds and tags the image with the full repository name
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '📤 Pushing image to Docker Hub...'
                // Retrieves credentials securely
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}",
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    
                    // Log in using the retrieved variables
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    
                    // Pushes the image using the fully qualified name
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                // All deployment logic must be within a 'steps' block
                script { // Using 'script' inside 'steps' allows for complex logic like sed
                    echo '⚙️ Deploying to Minikube...'

                    // CRITICAL FIX: Update certificate paths in the kubeconfig file 
                    // This resolves the 'no such file or directory' error for the jenkins user
                    sh "sed -i 's|/root/docker-project/minikube_data/.minikube|/var/lib/jenkins/.minikube|g' ${KUBECONFIG}"

                    // Update the deployment.yaml file to use the new Docker Hub image tag
                    sh "sed -i s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}| deployment.yaml"

                    echo '🚀 Applying Kubernetes deployment...'
                    sh 'kubectl apply -f deployment.yaml --validate=false --insecure-skip-tls-verify'
                    
                    // Wait for the deployment to fully rollout before finishing the stage
                    sh 'kubectl rollout status deployment/flask-app'
                }
            }
        }
    }

    // Define actions after all stages are complete
    post {
        success {
            echo "✅ Pipeline Success! Application deployed and rolled out."
        }
        failure {
            echo "❌ Pipeline FAILED! Check console output for deployment errors."
        }
    }
}
