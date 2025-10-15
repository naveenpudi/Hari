pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '460928920964.dkr.ecr.ap-south-1.amazonaws.com/hii'
        IMAGE_TAG = '17'  // increment this for new builds
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/naveenpudi/Hari.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${ECR_REPO}:${IMAGE_TAG} .'
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
                    sh 'aws eks update-kubeconfig --region ${AWS_REGION} --name exciting-monster-1760520550'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'AWS', region: "${AWS_REGION}") {
                    script {
                        sh '''
                            kubectl set image -f Deployment.yaml my-app=${ECR_REPO}:${IMAGE_TAG} --local -o yaml > temp-Deployment.yaml
                            kubectl apply -f temp-Deployment.yaml
                            kubectl apply -f Service.yaml
                            kubectl rollout status Deployment/my-app
                        '''
                    }
                }
            }
        }
    }
}
