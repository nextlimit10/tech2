pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '724772049461'  // ✅ Replace with your actual AWS Account ID
        AWS_REGION = 'us-west-2'         // ✅ Replace with your actual AWS region
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/web-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    deleteDir()  // Clean workspace before checkout
                    git credentialsId: 'github-credentials', branch: 'main', url: 'https://github.com/nextlimit10/tech2.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        set -e  # Fail immediately if any command fails
                        echo "Building Docker Image..."
                        docker build -t web-app .
                    '''
                    sh "echo 'Tagging Image: ${env.ECR_REPO}:latest'"
                    sh "docker tag web-app:latest ${env.ECR_REPO}:latest"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh '''
                        set -e  # Fail immediately if any command fails
                        echo "Logging into AWS ECR..."
                        aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.ECR_REPO}
                        echo "Pushing Docker Image..."
                        docker push ${env.ECR_REPO}:latest
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh '''
                        set -e  # Fail immediately if any command fails
                        echo "Deploying to EKS..."
                        kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }
}
