pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '724772049461'  // Replace with your actual AWS Account ID
        AWS_REGION = 'us-west-2'         // Replace with your AWS region
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
                    def ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/web-app"
                    sh 'sudo docker build -t web-app .'
                    sh "echo ECR_REPO=${ECR_REPO}"  // Debugging output
                    sh "docker tag web-app:latest ${ECR_REPO}:latest"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    def ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/web-app"
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
                    sh "docker push ${ECR_REPO}:latest"
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
