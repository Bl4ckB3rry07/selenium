pipeline {
    agent any

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Bl4ckB3rry07/selenium.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t mywebapp:${BUILD_NUMBER} .
                docker tag mywebapp:${BUILD_NUMBER} blackberry07/mywebapp:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push blackberry07/mywebapp:latest'
            }
        }

        stage('Start Minikube if not running') {
            steps {
                sh '''
                if ! minikube status | grep -q "apiserver: Running"; then
                    echo "Minikube is not running. Starting now..."
                    minikube start --driver=docker --memory=2048 --cpus=2
                fi
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
			minikube kubectl -- apply -f k8s/deployment.yaml
			minikube kubectl -- apply -f k8s/service.yaml
			SERVICE_URL=$(minikube service mywebapp-service --url)
			echo "Application URL: $SERVICE_URL"
			'''
            }
        }
    }
}
