pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/user/repository.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t web-app .'
                    sh 'docker tag web-app:latest <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/web-app:latest'
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region <AWS_REGION> | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com'
                    sh 'docker push <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/web-app:latest'
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
