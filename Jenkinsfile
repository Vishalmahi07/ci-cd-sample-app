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
                git 'https://github.com/Vishalmahi07/ci-cd-sample-app.git'
            }
        }

        stage('Install AWS CLI') {
            steps {
                sh 'sudo apt-get update'
                sh 'sudo apt-get install -y awscli'
            }
        }

        stage('Docker Login to ECR') {
            steps {
                sh """
                aws configure set aws_access_key_id $ACCESS_KEY
                aws configure set aws_secret_access_key $SECRET_KEY
                aws configure set default.region $AWS_REGION
                
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_REPO_NAME ."
                sh "docker tag $IMAGE_REPO_NAME:latest \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG"
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG"
            }
        }
    }
}
