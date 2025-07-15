pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/srikrishna206/jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('my-python-app')
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    docker.image('my-python-app').run('-d -p 5000:5000')
                }
            }
        }
    }
}
