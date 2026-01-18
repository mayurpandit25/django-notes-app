pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "mayurpandit25"
        IMAGE_NAME     = "notes-app"
        DOCKER_PAT_ID  = "dockerhub-credentials-id"
        KUBECONFIG_ID  = "kubeconfig"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mayurpandit25/django-notes-app.git'
            }
        }

        stage('Build & Test with Docker Compose') {
            steps {
                sh '''
                docker compose up -d --build
                docker compose down
                '''
            }
        }

        stage('Docker Login & Push Image') {
            steps {
                withCredentials([string(
                    credentialsId: "${DOCKER_PAT_ID}",
                    variable: "DOCKER_PAT"
                )]) {
                    sh '''
                    echo "$DOCKER_PAT" | docker login -u "$DOCKERHUB_USER" --password-stdin
                    docker build -t $DOCKERHUB_USER/$IMAGE_NAME:latest .
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(
                    credentialsId: "${KUBECONFIG_ID}",
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                    export KUBECONFIG=$KUBECONFIG
                    kubectl apply -f k8s/.
                    '''
                }
            }
        }
    }
}
