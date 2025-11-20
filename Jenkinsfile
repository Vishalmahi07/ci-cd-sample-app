pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "109427006764"
        AWS_REGION = "ap-south-1"
        IMAGE_REPO_NAME = "ci-cd-sample-app"
        IMAGE_TAG = "latest"
        ACCESS_KEY = credentials('aws-access-key')
        SECRET_KEY = credentials('aws-secret-key')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Vishalmahi07/ci-cd-sample-app.git'
            }
        }

        stage('Install Docker & AWS CLI v2') {
            steps {
                sh '''
                sudo apt-get update -y
                sudo apt-get install -y docker.io unzip curl
                sudo systemctl start docker
                sudo systemctl enable docker
                sudo usermod -aG docker jenkins || true
                curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                unzip -o awscliv2.zip
                sudo ./aws/install || true
                '''
            }
        }

        stage('Configure AWS & Docker Login to ECR') {
            steps {
                sh '''
                export AWS_ACCESS_KEY_ID=$ACCESS_KEY
                export AWS_SECRET_ACCESS_KEY=$SECRET_KEY
                export AWS_DEFAULT_REGION=$AWS_REGION

                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_REPO_NAME .
                docker tag $IMAGE_REPO_NAME:latest \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                docker push \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}

