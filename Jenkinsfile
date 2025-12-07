pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "851725169930.dkr.ecr.ap-south-1.amazonaws.com/backend-service"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-credentials']]) {
                    sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t backend-service ."
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh "docker tag backend-service:latest $ECR_REPO:latest"
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push $ECR_REPO:latest"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@<EC2_PUBLIC_IP> '
                        docker pull $ECR_REPO:latest &&
                        docker stop backend || true &&
                        docker rm backend || true &&
                        docker run -d -p 5000:5000 --name backend $ECR_REPO:latest
                    '
                    """
                }
            }
        }
    }
}