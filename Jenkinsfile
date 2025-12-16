pipeline {
    agent any

    environment {
        COMPOSE_FILE = 'docker-compose.yml'
        // change this to your kubeconfig path
        KUBECONFIG = 'C:\\Users\\dongk\\.kube\\config'
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

        // stage('Deploy to Kubernetes') {
        //     steps {
        //         echo "Building images for Kubernetes and applying manifests..."

        //         bat """
        //         cd %WORKSPACE%\\bank-api
        //         docker build -t bank-api:latest .
        //         """

        //         bat """
        //         cd %WORKSPACE%\\bank-web
        //         docker build --build-arg REACT_APP_API_URL=http://bank-api:9090/api/v1 -t bank-web:latest .
        //         """

        //         bat """
        //         cd %WORKSPACE%
        //         kubectl config use-context docker-desktop
        //         kubectl apply -f k8/namespace.yaml --validate=false
        //         kubectl apply -f k8/bankapi.yaml --validate=false
        //         kubectl apply -f k8/bankweb.yaml --validate=false
        //         """

        //         echo "Kubernetes resources after deploy:"
        //         bat """
        //         kubectl get pods -n bank
        //         kubectl get svc -n bank
        //         """
        //     }
        // }
    }

    post {
        success {
            echo " Deployment successful!"
            echo "- Docker Compose:"
            echo "    Web: http://localhost:3000"
            echo "    API: http://localhost:9090"
            echo ""
            echo "- Kubernetes (Docker Desktop):"
            echo "    Use port-forward in a terminal:"
            echo "      kubectl port-forward svc/bank-web -n bank 3000:80"
            echo "      kubectl port-forward svc/bank-api -n bank 9090:9090"
        }
        failure {
            echo "Deployment failed check console log for details."
        }
    }
}