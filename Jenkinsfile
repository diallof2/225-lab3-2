pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'diallof2/lab3-app'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/diallof2/225-lab3-2.git'
        KUBECONFIG = credentials('roseaw-225')
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Checking out code from GitHub..."
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "📤 Pushing Docker image to DockerHub..."
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying to Kubernetes..."
                script {
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment.yaml"
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f ingress.yaml"
                }
            }
        }

        stage('Verify Kubernetes Resources') {
            steps {
                echo "🔍 Verifying deployed services..."
                sh "kubectl get all"
            }
        }
    }

    post {
        success {
            slackSend([color: "good", message: "✅ Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}"])
        }
        unstable {
            slackSend([color: "warning", message: "⚠️ Build Unstable: ${env.JOB_NAME} #${env.BUILD_NUMBER}"])
        }
        failure {
            slackSend([color: "danger", message: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"])
        }
    }
}
