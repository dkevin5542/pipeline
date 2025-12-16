pipeline {
    agent any

    environment {
        COMPOSE_FILE = 'docker-compose.yml'

        // Repo directories
        TF_DIR      = 'terraform'

        // change this to your kubeconfig path
        // KUBECONFIG = 'C:\\Users\\dongk\\.kube\\config'
    }

    stages {
        // ------------------------------------------------------------
        // CHECKOUT SOURCE CODE
        // ------------------------------------------------------------
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        /*
        // STAGE 1: PROVISION INFRASTRUCTURE (TERRAFORM)
        // This stage is commented out. Uncomment it and install the 'terraform' CLI tool to enable.
        stage('Provision Infrastructure (Terraform)') {
            steps {
                dir(TF_DIR) {
                    sh 'terraform init'
                    sh 'terraform validate'
                    sh 'terraform apply -auto-approve' 
                }
            }
        }
        */

        // ------------------------------------------------------------
        // BUILD & TEST APPLICATIONS; REMOVE OLD CONTAINERS
        // ------------------------------------------------------------
        stage('Build & Deploy with Docker Compose') {
            steps {
                echo "Stopping and removing any existing containers..."

                bat """
                cd %WORKSPACE%
                docker-compose -f %COMPOSE_FILE% down || exit /b 0
                docker compose down --remove-orphans || exit /b 0
                """

                echo "Building and starting containers with docker-compose..."
                bat """
                cd %WORKSPACE%
                docker-compose -f %COMPOSE_FILE% up -d --build
                """
            }
        }

        stage('Show Running Containers') {
            steps {
                bat "docker ps"
            }
        }

        // ------------------------------------------------------------
        // DEPLOY TO KUBERNETES AND VERIFY (DOCKER DESKTOP)
        // ------------------------------------------------------------
        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                kubectl config use-context docker-desktop
                kubectl apply -f k8/namespace.yaml
                kubectl apply -f k8/bank-api.yaml
                kubectl apply -f k8/bank-web.yaml
                kubectl rollout status deployment/bank-api -n bank
                kubectl rollout status deployment/bank-web -n bank
                kubectl get pods -n bank
                kubectl get svc -n bank
                '''
            }
        }
    }

    post {
        success {
            echo " Deployment successful!"
            echo "- Docker Compose:"
            echo "    Web: http://localhost:3000"
            echo "    API: http://localhost:9090"
        }
        failure {
            echo "Deployment failed check console log for details."
        }
    }
}