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
                withAWS(credentials: 'aws-cred-id', region: "${AWS_REGION}") {
                    script {
                        sh '''
                        echo "Checking for existing ECR repo..."
                        if ! aws ecr describe-repositories --repository-names $REPO_NAME > /dev/null 2>&1; then
                            echo "ECR repo not found. Creating..."
                            aws ecr create-repository --repository-name $REPO_NAME
                        else
                            echo "ECR repo already exists."
                        fi

                        export ECR_REPO=$(aws ecr describe-repositories --repository-names $REPO_NAME --query "repositories[0].repositoryUri" --output text)
                        echo "ECR_REPO=$ECR_REPO" >> env.properties
                        '''
                    }
                }
            }
        }

        stage('Load ECR Repo Info') {
            steps {
                script {
                    def props = readProperties file: 'env.properties'
                    env.ECR_REPO = props['ECR_REPO']
                    echo "Using ECR repository: ${ECR_REPO}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ECR_REPO}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-cred-id', region: "${AWS_REGION}") {
                    sh '''
                    aws ecr get-login-password \
                        | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh "docker push ${ECR_REPO}:${BUILD_NUMBER}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'aws-cred-id', region: "${AWS_REGION}") {
                    sh '''
                    aws eks update-kubeconfig --name $CLUSTER_NAME
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
