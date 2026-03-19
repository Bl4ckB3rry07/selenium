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
                // Keep the Hub push if you want it as a backup
                sh 'docker push blackberry07/mywebapp:latest'
                
                // ADD THIS: Load the image directly into Minikube
                echo "Loading image into Minikube node..."
                sh 'minikube image load blackberry07/mywebapp:latest'
            }
        }

        stage('Start Minikube if not running') {
            steps {
                sh '''
		# 1. Force clear any system-wide locks
			sudo rm -rf /tmp/minikube-locks

			# 2. Check if minikube is already healthy
			if ! minikube status | grep -q "apiserver: Running"; then
			    echo "Minikube is not running or is unhealthy. Starting..."
			    
			    # 3. Use 'minikube delete' if it exists but is broken 
			    # (common after permission-denied crashes)
			    minikube delete || true
			    
			    # 4. Start fresh
			    minikube start --driver=docker --memory=2048 --cpus=2
			else
			    echo "Minikube is already running and healthy."
			fi
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
			sh '''
				minikube kubectl -- apply -f k8s/deployment.yaml
				minikube kubectl -- apply -f k8s/service.yaml
				
				# Force a restart to pick up the newly loaded image
				minikube kubectl -- rollout restart deployment/mywebapp-deployment
				
				echo "Waiting for pods..."
				minikube kubectl -- rollout status deployment/mywebapp-deployment --timeout=180s
				
				SERVICE_URL=$(minikube service mywebapp-service --url)
				echo "Application URL: $SERVICE_URL"
				'''
            }
        }
    }
}
