pipeline {
    agent any

    environment {
        COMPOSE_FILE = 'docker-compose.yml'
        // change this to your kubeconfig path
        // KUBECONFIG = 'C:\\Users\\dongk\\.kube\\config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

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