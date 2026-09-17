pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '583749796090'
        ECR_REPOSITORY = 'devops-cicd-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build \
                -t ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password \
                --region ${AWS_REGION} | \
                docker login \
                --username AWS \
                --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                docker push \
                ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                kubectl set image deployment/devops-cicd-demo \
                devops-cicd-demo=${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl rollout status deployment/devops-cicd-demo'
                sh 'kubectl get pods'
            }
        }
    }
}
