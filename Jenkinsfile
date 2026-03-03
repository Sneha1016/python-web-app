pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "python-web-app"
        ACCOUNT_ID = "349036691344"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin \
                    ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh """
                        docker tag ${ECR_REPO}:${IMAGE_TAG} \
                        ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}

                        docker push \
                        ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy via SSM') {
            steps {
                script {
                    sh """
                        aws ssm send-command \
                        --instance-ids i-0cf49a28a05032be4 \
                        --document-name "AWS-RunShellScript" \
                        --parameters commands="docker pull ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}; docker stop app || true; docker rm app || true; docker run -d --name app -p 5000:5000 ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}" \
                        --region ${AWS_REGION}
                    """
                }
            }
        }
    }
}
