pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "109427006764"
        AWS_REGION = "ap-south-1"
        IMAGE_REPO_NAME = "ci-cd-sample-app"
        IMAGE_TAG = "latest"
        ACCESS_KEY = credentials('aws-access-key-id')
        SECRET_KEY = credentials('aws-secret-access-key')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Vishalmahi07/ci-cd-sample-app.git'
            }
        }

        stage('Docker Login to ECR') {
            steps {
                sh '''
                aws configure set aws_access_key_id $ACCESS_KEY
                aws configure set aws_secret_access_key $SECRET_KEY
                aws configure set default.region $AWS_REGION

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

        stage('Deploy to EC2') {
            steps {
                sh '''
                docker rm -f app || true
                docker pull \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
                docker run -d -p 3000:3000 --name app \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
