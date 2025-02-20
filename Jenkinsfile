pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/nextlimit10/tech2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t web-app .'
                    sh 'docker tag web-app:latest 724772049461.dkr.ecr.us-west-2.amazonaws.com/web-app:latest'
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 724772049461.dkr.ecr.us-west-2.amazonaws.com'
                    sh 'docker push 724772049461.dkr.ecr.us-west-2.amazonaws.com/web-app:latest'
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
