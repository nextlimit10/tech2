Tech Challenge 2
Challenge 2: Continuous Deployment with Jenkins, Docker, and AWS EKS
Objective
Deploy a web application using Docker, orchestrate it with AWS EKS, and set up a continuous deployment pipeline using Jenkins.
Rules of Submission
This challenge must be completed within 72 hours.
Code and assets must be version controlled using GitHub.
Submission must include a GitHub repository link.
The GitHub project should have a README file with instructions for installation and running the project.
Ensure the private repository is shared with your mentor.
Overview
This challenge assesses the candidate’s ability to use Docker, Jenkins, and AWS EKS for deploying applications and setting up a CI/CD pipeline.
Requirements
Web Application
Create a simple web application (e.g., Node.js or Python Flask) that displays "Hello, World!".
Docker
Dockerize the application and create a Dockerfile.
Kubernetes
Use Terraform to provision an EKS cluster on AWS.
Deploy the Dockerized application to the EKS cluster.
Jenkins
Set up a Jenkins server.
Create a Jenkins pipeline that:
Builds the Docker image and pushes it to Amazon ECR.
Deploys the application to the EKS cluster using kubectl.
Documentation
Include a detailed README file with:
Instructions for setting up the environment.
Steps to deploy the application.
Explanation of the Terraform code and Jenkins pipeline.
Submission Requirements
A private GitHub repository with all the code and configuration files.
Provide the URL of the deployed application.
Share the GitHub repository with your mentor.
Evaluation Criteria
You will be evaluated based on:
The functionality of the hosted application.
The proper setup and execution of the CI/CD pipeline.
The quality and organization of the Terraform and Jenkins code.


Step-by-Step Guide: Deploying a Web Application using Jenkins, Docker, and AWS EKS
Prerequisites
Before starting, ensure you have the following:
AWS Account with necessary IAM permissions
GitHub Account (for code repository)
Terraform Installed (terraform -v)
Docker Installed (docker -v)
AWS CLI Installed (aws --version)
Kubectl Installed (kubectl version)
Jenkins Installed (locally or AWS-managed)
Amazon ECR Setup (to store Docker images)

Step 1: Develop a Simple Web Application
1.1 Create a Node.js (Express) or Python (Flask) App
Create a directory for the project:
bash

mkdir web-app && cd web-app

For Node.js (Express)
Initialize a Node.js project:
bash

npm init -y


Install Express:
bash

npm install express


Create app.js:
javascript

const express = require("express");
const app = express();
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
    res.send("Hello, World!");
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});


Test locally:
bash

node app.js


For Python (Flask)
Install Flask:
bash
pip install flask


Create app.py:
python

from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "Hello, World!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)


Run locally:
bash
python app.py



Step 2: Dockerize the Application
2.1 Create a Dockerfile
For Node.js
dockerfile

FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
CMD ["node", "app.js"]
EXPOSE 3000

For Python (Flask)
dockerfile

FROM python:3.9
WORKDIR /app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
EXPOSE 5000

2.2 Build and Run the Docker Image
bash

docker build -t web-app .
docker run -p 3000:3000 web-app  # Replace port for Flask (5000:5000)




Step 3: Set Up AWS EKS using Terraform
3.1 Install Terraform and AWS CLI
Ensure you have AWS CLI configured (Skip this step if AWS is already configured):
bash

aws configure

3.2 Create Terraform Configuration
Create main.tf for provisioning EKS Cluster  (KEEP IN MIND MY TERRAFORM IS CONFIGURED TO MY REGION AND AZ SO YOU WILL HAVE TO MAKE CHANGES FOR THE CODE TO PROPERLY WORK!!):
hcl

provider "aws" {
  region = "us-west-2"
}


resource "aws_vpc" "eks_vpc" {
  cidr_block = "10.0.0.0/16"
}


resource "aws_subnet" "public_subnet_1" {
  vpc_id            = aws_vpc.eks_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-west-2a"
  map_public_ip_on_launch = true
}


resource "aws_subnet" "public_subnet_2" {
  vpc_id            = aws_vpc.eks_vpc.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-west-2b"
  map_public_ip_on_launch = true
}


resource "aws_iam_role" "eks_role" {
  name = "eks-cluster-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = { Service = "eks.amazonaws.com" }
      Action = "sts:AssumeRole"
    }]
  })
}


resource "aws_iam_role_policy_attachment" "eks_cluster_policy" {
  role       = aws_iam_role.eks_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
}


resource "aws_iam_role_policy_attachment" "node_group_AmazonEKSWorkerNodePolicy" {
  role       = aws_iam_role.node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
}


resource "aws_iam_role_policy_attachment" "node_group_AmazonEC2ContainerRegistryReadOnly" {
  role       = aws_iam_role.node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
}


resource "aws_iam_role_policy_attachment" "node_group_AmazonEKS_CNI_Policy" {
  role       = aws_iam_role.node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
}


resource "aws_eks_cluster" "eks_cluster" {
  name     = "my-cluster"
  role_arn = aws_iam_role.eks_role.arn
  vpc_config {
    subnet_ids = [aws_subnet.public_subnet_1.id, aws_subnet.public_subnet_2.id]
    endpoint_public_access = true
    endpoint_private_access = true
  }
}


resource "aws_iam_role" "node_role" {
  name = "eks-node-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
      Action = "sts:AssumeRole"
    }]
  })
}


resource "aws_internet_gateway" "eks_igw" {
  vpc_id = aws_vpc.eks_vpc.id
}


resource "aws_route_table" "eks_route_table" {
  vpc_id = aws_vpc.eks_vpc.id


  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.eks_igw.id
  }
}


resource "aws_route_table_association" "subnet_1_association" {
  subnet_id      = aws_subnet.public_subnet_1.id
  route_table_id = aws_route_table.eks_route_table.id
}


resource "aws_route_table_association" "subnet_2_association" {
  subnet_id      = aws_subnet.public_subnet_2.id
  route_table_id = aws_route_table.eks_route_table.id
}






resource "aws_security_group" "eks_sg" {
  vpc_id = aws_vpc.eks_vpc.id


  # Allow nodes to talk to each other
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }


  # Allow nodes to communicate with the EKS API server
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  # Allow all outbound traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}




resource "aws_eks_node_group" "node_group" {
  cluster_name    = aws_eks_cluster.eks_cluster.name
  node_group_name = "my-node-group"
  node_role_arn   = aws_iam_role.node_role.arn
  subnet_ids      = [aws_subnet.public_subnet_1.id, aws_subnet.public_subnet_2.id]
  instance_types  = ["t3.medium"]


  scaling_config {
    desired_size = 2
    max_size     = 3
    min_size     = 1
  }
}





3.3 Initialize and Apply Terraform
bash

terraform init
terraform apply -auto-approve


3.4 Push to GitHub
bash
CopyEdit
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/user/repository.git
git push -u origin main
3.5 Configure kubectl to Connect to EKS

Ensure Your IAM User Has ECR Permissions
Your AWS IAM user must have the following policies attached: ✅ AmazonEC2ContainerRegistryFullAccess
To attach this permission, run:
sh

aws iam attach-user-policy --user-name your-iam-user --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess

Run the following command to create an ECR repository named web-app:
sh

aws ecr create-repository --repository-name web-app --region us-west-2

Example output:
json

{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-west-2:123456789012:repository/web-app",
        "registryId": "123456789012",
        "repositoryName": "web-app",
        "repositoryUri": "123456789012.dkr.ecr.us-west-2.amazonaws.com/web-app"
    }
}


Note down the repositoryUri because we'll use it to push our image.
Before pushing images, Docker must authenticate with Amazon ECR.
Run:
sh
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com



Expected output:
plaintext
Login Succeeded
First, confirm that the Docker image exists:

sh

docker images

Example output:
plaintext

REPOSITORY       TAG        IMAGE ID       CREATED         SIZE
web-app       latest     d3b2c3a5f7b8   5 minutes ago   120MB

Now, tag the image with the repository URI:
sh

docker tag web-app:latest 123456789012.dkr.ecr.us-west-2.amazonaws.com/web-app:latest


Verify the tag:
sh

docker images

Output should now include:
plaintext

REPOSITORY                                                TAG        IMAGE ID       CREATED         SIZE
web-app                                               latest     d3b2c3a5f7b8   5 minutes ago   120MB
123456789012.dkr.ecr.us-west-2.amazonaws.com/web-app  latest     d3b2c3a5f7b8   5 min

Make sure to configure based on your terraform code, region etc. k

Now, push the tagged image to Amazon ECR:
sh

docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/web-app:latest


This will take a few moments. Expected output:
plaintext

The push refers to repository [123456789012.dkr.ecr.us-west-2.amazonaws.com/web-app]
...
latest: digest: sha256:abcdef1234567890 size: 1234

bash
CopyEdit
aws eks update-kubeconfig --name my-eks-cluster --region us-west-2




To confirm the image was successfully uploaded, run:
sh

aws ecr list-images --repository-name web-app --region us-west-2



Step 4: Deploy Application to EKS
4.1 Create Kubernetes Deployment
Create deployment.yaml:
yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-app
          image: <ECR_REPO_URL>/my-app:latest
          ports:
            - containerPort: 3000



Also add the service.yaml file:
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000  # Change to 5000 if using Flask

Considerations:
Use ConfigMaps and Secrets instead of hardcoded environment variables.
Set up horizontal pod autoscaling for better performance.

4.2 Apply Deployment
bash
CopyEdit
kubectl apply -f deployment.yaml
Kubectl apply -f service.yaml


Steps for putting Docker in EC2 and get the jenkins container to use Jenkins in the EC2
Run to following commands to get docker in the EC2:
sudo apt update && sudo apt-get update

sudo usermod -aG docker $USER
sudo apt install docker.io


Run the following command to get the docker container that has jenkins in it remotely:

docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts

Check if it the Docker container has Jenkins:
docker ps -a

docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts
Check if the docker container was build and copy container id:
docker ps -a

Run the following command to get into the docker container:
docker exec -it <container id> bash

The docker container needs to have the following to be able to run the pipeline:
Aws-cli
Docker
Git access
Kubertl

Research what commands you need based on the version of the machine you are running and make sure to run commands like docker -- version to check if it was properly installed.

While still in the docker container run this command to get the jenkins password:
cat /var/jenkins_home/secrets/initialAdminPassword

Finish configuring Jenkins
Copy the password that you get from the cat command and paste it inside the Jenkins UI under "Administrator Password" and click continue


Click on "Install Suggested Plugins" and the plugins for install for your Jenkins server. We want you to use the admin account so don't create a user. The username will be "admin" and the password will be the password you just used to enter jenkins (The Default Password). Go ahead and save this information for the future. (DO THIS NOW!!!!!)


On the "Instance Configuration" page that comes up afterwards, don't change anything and just click on "Save and Finish"


Step 5: Set Up Jenkins CI/CD Pipeline

Step 5.1: Install Necessary Plugins
Open Jenkins: Launch Jenkins in your web browser (typically available at http://Your_EC2_IP_Address:8080).
Access Manage Jenkins: From the Jenkins dashboard, navigate to Manage Jenkins > Manage Plugins.
Install Plugins: Go to the Available Plugins, and use the search box to find the following plugins:
Docker Pipeline plugin: For building and pushing Docker images from Jenkins pipelines.


Amazon ECR: For generating authentication token from Amazon Credentials to access Amazon ECR
Pipeline Stage step: Just a nice plugin that provides a GUI when building the pipeline to see where errors were taking place.
Kubernetes CLI Plugin: Allows you to configure kubectl to interact with Kubernetes clusters from within your jobs.


Select these plugins and click Install.
Step 2: Add Credentials
    Generating a Personal Access Token (PAT) on GitHub
Login to GitHub:
Sign in to your GitHub account.


Access Settings: Click on your profile photo in the upper-right corner of any page, then click Settings.


Developer Settings: Scroll down the sidebar until you find Developer settings and click on it.


Personal Access Tokens: Click on Personal access tokens, then click on the Generate new token button.


Set Token Description: Give your token a descriptive name in the Note field to remember where it's used.


Select Scopes: Choose the scopes or permissions you'd like to grant this token. For Jenkins integration, you might need to select scopes such as repo, admin:repo_hook, etc., depending on what operations Jenkins needs to perform.


Generate Token: Click the Generate token button at the bottom of the page. Make sure to copy your new personal access token now!!!!

	AWS Credentials:
Navigate to Manage Jenkins > Credentials > System > Global credentials (unrestricted) > Add Credentials
Choose Username and password from the Kind dropdown
Enter your AWS Access Key ID and Secret Access Key. Assign an ID (e.g., "aws-credentials") and description for identification.

Docker Credentials and ECR Credentials:
Still in the credentials section, add a new credetnial of type Username with password
For Username, enter your AWS access key. For Password, enter your AWS secret access key. This will be used to log into ECR. Give it an ID (e.g., 'aws-credentials').
Git credentials:
Kind: Select Username with password.
Scope: Choose Global (Jenkins, nodes, items, all child items, etc).
Username: Enter your GitHub username.
Password: Paste the Personal Access Token (PAT) you generated on GitHub.
ID : Enter an identifier for your credentials (e.g., github-crednetials). THIS WILL BE THE SAME IN THE PIPELINE CODE SO IT CAN COMMUNICATE WITH GITHUB!!!
Description (optional): Provide a description for these credentials (e.g., "GitHub PAT for Jenkins"). Save Credentials: Click the OK button to save your credentials.

Create New Item:
On the Jenkins dashboard, click on “New Item” at the top left.
Enter a name for your pipeline in the “Enter an item name” field.
Select “Pipeline” from the list of options.
Click “OK” to proceed.
Define Pipeline:
Pipeline script (However, Pipeline script from SCM is recommended)
Select “Pipeline script”.
Code the following pipeline code in the next page in the script

5.2 Configure Jenkins Pipeline
Create a Jenkinsfile:
groovy
CopyEdit
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
                        #!/bin/bash
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
                        #!/bin/bash
                        set -e  # Fail immediately if any command fails
                        echo "Checking AWS CLI installation..."
                        aws --version  # ✅ Verify AWS CLI is installed  
                        
                        echo "Exporting Variables..."
                        export AWS_REGION="${AWS_REGION}"
                        export ECR_REPO="${ECR_REPO}"
                        
                        echo "Logging into AWS ECR..."
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO

                        echo "Pushing Docker Image..."
                        docker push $ECR_REPO:latest
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh '''
                        #!/bin/bash
                        set -e  # Fail immediately if any command fails
                        echo "Deploying to EKS..."
                        aws eks update-kubeconfig --region us-west-2 --name my-cluster
                        kubectl apply -f deployment.yaml
		Kubectl appy -f service.yaml
                    '''
                }
            }
        }
    }
}




5.3 Configure Jenkins Job
Open Jenkins Dashboard.
Click New Item → Pipeline
Enter GitHub Repository URL.
Save and click Build Now.

Step 6: Document & Submit
6.1 Create README.md
Include:
Project Setup Instructions
Terraform, Docker, Kubernetes, and Jenkins Pipeline Steps
Common Errors & Fixes
Deployed URL


6.2 Share Submission
Submit GitHub repository link.
Provide Deployed URL.






