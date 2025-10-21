pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '650576187890.dkr.ecr.ap-south-1.amazonaws.com/three-tier-application'
        IMAGE_TAG = "vprofile-${env.BUILD_NUMBER}"
        EC2_USER = 'ubuntu'
        EC2_HOST = '3.109.184.223'  
        SSH_KEY = 'ec2-key'     
        }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Aalu14/three-tier-application.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Login & Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ["${SSH_KEY}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO} &&
                            cd /home/ubuntu/vprofile-project &&
                            docker compose pull &&
                            docker compose up -d
                        "
                    """
                }
            }
        }
    }
}