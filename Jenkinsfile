pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        REPO_NAME = 'madhu'
        CLUSTER_NAME = 'eksctl-exciting-monster-1760520550-cluster'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/naveenpudi/Hari.git'
            }
        }

        stage('Create ECR Repo (if not exists)') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS'
                ]]) {
                    script {
                        env.ECR_REPO = sh(
                            script: '''
                                if ! aws ecr describe-repositories --repository-names $REPO_NAME --region $AWS_REGION > /dev/null 2>&1; then
                                    echo "ECR repo not found. Creating..."
                                    aws ecr create-repository --repository-name $REPO_NAME --region $AWS_REGION
                                else
                                    echo "ECR repo already exists."
                                fi
                                aws ecr describe-repositories --repository-names $REPO_NAME --region $AWS_REGION --query "repositories[0].repositoryUri" --output text
                            ''',
                            returnStdout: true
                        ).trim()
                        echo "Using ECR repository: ${env.ECR_REPO}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.ECR_REPO}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS'
                ]]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh "docker push ${env.ECR_REPO}:${BUILD_NUMBER}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS'
                ]]) {
                    sh '''
                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                    sed -i "s|IMAGE_PLACEHOLDER|${ECR_REPO}:${BUILD_NUMBER}|g" Deployment.yaml
                    kubectl apply -f Deployment.yaml
                    kubectl apply -f Service.yaml
                    kubectl rollout status deployment/hari-app
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment to EKS successful!"
        }
        failure {
            echo "❌ Deployment failed. Check logs."
        }
    }
}
