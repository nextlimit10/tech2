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
                    def ecrRepo = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/web-app"
                    sh 'sudo docker build -t web-app .'
                    sh "echo AWS_ACCOUNT_ID=${env.AWS_ACCOUNT_ID} AWS_REGION=${env.AWS_REGION} ECR_REPO=${ecrRepo}"  // Debugging
                    sh "docker tag web-app:latest ${ecrRepo}:latest"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    def ecrRepo = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/web-app"
                    sh "aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${ecrRepo}"
                    sh "docker push ${ecrRepo}:latest"
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
