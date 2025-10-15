pipeline {
    agent any

    environment {
        ECR_REPO = "460928920964.dkr.ecr.ap-south-1.amazonaws.com/hii"
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_REGION = "ap-south-1"
        KUBE_CLUSTER = "exciting-monster-1760520550"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/naveenpudi/Hari.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }

        stage('Login to ECR & Push') {
            steps {
                withAWS(credentials: 'AWS', region: "${AWS_REGION}") {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Configure kubectl') {
            steps {
                withAWS(credentials: 'AWS', region: "${AWS_REGION}") {
                    sh 'aws eks update-kubeconfig --region ${AWS_REGION} --name ${KUBE_CLUSTER}'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'AWS', region: "${AWS_REGION}") {
                    script {
                        sh '''
                            cd Hari
                            kubectl set image -f deployment.yaml my-app=${ECR_REPO}:${IMAGE_TAG} --local -o yaml > temp-deployment.yaml
                            kubectl apply -f temp-deployment.yaml
                            kubectl apply -f service.yaml
                            kubectl rollout status deployment/my-app
                        '''
                    }
                }
            }
        }
    }
}
