pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '460928920964.dkr.ecr.ap-south-1.amazonaws.com/hii'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CLUSTER = 'exciting-monster-1760520550'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/naveenpudi/Hari.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $ECR_REPO:$IMAGE_TAG ."
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push $ECR_REPO:$IMAGE_TAG"
            }
        }

        stage('Configure kubectl') {
            steps {
                script {
                    sh "aws eks --region $AWS_REGION update-kubeconfig --name $KUBE_CLUSTER"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                    kubectl set image deployment/my-app my-app=$ECR_REPO:$IMAGE_TAG --record
                    kubectl rollout status deployment/my-app
                    """
                }
            }
        }
    }
}
