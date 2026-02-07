pipeline {
    agent any

    environment {
        AWS_REGION     = "us-east-1"
        AWS_ACCOUNT_ID = "017820689407"
        ECR_REPO       = "user-service"
        ECR_URI        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        ECS_CLUSTER    = "rrr-cluster"
        ECS_SERVICE    = "userservice-task-service-0qigdupy"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                  docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                  docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:latest
                """
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'awsid'
                ]]) {
                    sh """
                      aws ecr get-login-password --region ${AWS_REGION} | \
                      docker login --username AWS --password-stdin \
                      ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                  docker push ${ECR_URI}:${IMAGE_TAG}
                  docker push ${ECR_URI}:latest
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'awsid'
                ]]) {
                    sh """
                      aws ecs update-service \
                        --cluster ${ECS_CLUSTER} \
                        --service ${ECS_SERVICE} \
                        --force-new-deployment \
                        --region ${AWS_REGION}
                    """
                }
            }
        }
    }

    post {
        always {
            sh '''
              if command -v docker >/dev/null 2>&1; then
                docker image prune -f
              fi
            '''
        }
    }
}
