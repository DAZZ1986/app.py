pipeline {
	agent any

	environment {
		// Имя вашего Docker-образа
		DOCKER_IMAGE = "my-pet-app:${BUILD_ID}"
		// Имя вашего k8s деплоймента
		DEPLOYMENT_NAME = "my-pet-app-deployment"
		// Имя сервиса
		SERVICE_NAME = "my-pet-app-service"
	}

	stages {
		stage('Checkout') {
			steps {
				git branch: 'main', url: 'https://github.com/DAZZ1986/app.py.git'
			}
		}

		stage('Build Docker Image') {
			steps {
				script {
					// Строим образ локально в Docker Мини-куба
					sh "docker build -t ${DOCKER_IMAGE} ."
				}
			}
		}

		stage('Deploy to Kubernetes') {
			steps {
				script {
					// Создаем деплоймент и выставляем порт 5000
					sh """
						kubectl create deployment ${DEPLOYMENT_NAME} --image=${DOCKER_IMAGE} --dry-run=client -o yaml | kubectl apply -f -
						kubectl set image deployment/${DEPLOYMENT_NAME} python=${DOCKER_IMAGE}
						kubectl expose deployment ${DEPLOYMENT_NAME} --port=80 --target-port=5000 --type=NodePort --dry-run=client -o yaml | kubectl apply -f -
					"""
				}
			}
		}
	}

	post {
		success {
			echo 'Pipeline completed successfully!'
			echo "Deployment successful! Check pods with: kubectl get pods"
			script {
				def serviceUrl = sh(script: "minikube service ${SERVICE_NAME} --url", returnStdout: true).trim()
				echo "Your app is available at: ${serviceUrl}"
			}
		}
	}
}