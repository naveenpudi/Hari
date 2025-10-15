pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '460928920964.dkr.ecr.ap-south-1.amazonaws.com/hii'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CLUSTER = 'exciting-monster-1760520550'
        DEPLOY_PATH = '/home/jenkins/deploy' // path where deployment.yaml and service.yaml are stored
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/naveenpudi/Hari.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $ECR_REPO:$IMAGE_TAG ."
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-jenkins-credential-id', region: "${AWS_REGION}") {
                    sh "aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REPO"
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
                withAWS(credentials: 'aws-jenkins-credential-id', region: "${AWS_REGION}") {
                    sh "aws eks --region $AWS_REGION update-kubeconfig --name $KUBE_CLUSTER"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    // Update the image in the deployment file dynamically
                    sh "kubectl set image -f $DEPLOY_PATH/deployment.yaml my-app=$ECR_REPO:$IMAGE_TAG --record"
                    
                    // Apply the existing deployment and service YAML files
                    sh "kubectl apply -f $DEPLOY_PATH/deployment.yaml"
                    sh "kubectl apply -f $DEPLOY_PATH/service.yaml"
                    
                    // Wait for rollout to complete
                    sh "kubectl rollout status deployment/my-app"
                }
            }
        }
    }
}
