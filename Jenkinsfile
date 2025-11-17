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
                git branch: 'main', url:'https://github.com/afroz2004905/hello.git'
                sh 'ls -l'
            }
        }

