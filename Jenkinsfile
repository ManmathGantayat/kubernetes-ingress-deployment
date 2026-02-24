pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ACCOUNT_ID = "426192960096"
        ECR_REGISTRY = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        REPO_NAME = "kubernetes-ingress-deployment"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ManmathGantayat/kubernetes-ingress-deployment.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t main .
                docker build -t aws ./aws
                docker build -t azure ./azure
                docker build -t gcp ./gcp
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} \
                | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag & Push Images to ECR') {
            steps {
                sh '''
                docker tag main:latest   ${ECR_REGISTRY}/main:latest
                docker tag aws:latest    ${ECR_REGISTRY}/aws:latest
                docker tag azure:latest  ${ECR_REGISTRY}/azure:latest
                docker tag gcp:latest    ${ECR_REGISTRY}/gcp:latest

                docker push ${ECR_REGISTRY}/main:latest
                docker push ${ECR_REGISTRY}/aws:latest
                docker push ${ECR_REGISTRY}/azure:latest
                docker push ${ECR_REGISTRY}/gcp:latest
                '''
            }
        }

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                sh '''
                kubectl apply -f k8s-files/
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Access app via ALB DNS / Route53 domain"
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs."
        }
    }
}
