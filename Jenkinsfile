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
                    sh 'sudo docker build -t web-app .'
                    sh "echo AWS_ACCOUNT_ID=${env.AWS_ACCOUNT_ID} AWS_REGION=${env.AWS_REGION} ECR_REPO=${env.ECR_REPO}"  // Debugging
                    sh "docker tag web-app:latest ${env.ECR_REPO}:latest"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.ECR_REPO}"
                    sh "docker push ${env.ECR_REPO}:latest"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}
