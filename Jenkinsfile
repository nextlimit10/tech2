pipeline { 
    agent any 
    environment { 
        AWS_REGION = 'us-west-2' 
        ECR_REPO = '<account_id>.dkr.ecr.us-west-2.amazonaws.com/tech-challenge2-app' 
        CLUSTER_NAME = 'my-cluster' 
    } 
    stages { 
        stage('Build and Push Docker Image') { 
            steps { 
                sh 'docker build -t $ECR_REPO .' 
                sh 'docker push $ECR_REPO' 
            } 
        } 
        stage('Deploy to EKS') { 
            steps { 
                sh 'kubectl apply -f deployment.yaml' 
                sh 'kubectl apply -f service.yaml' 
            } 
        } 
    } 
}
