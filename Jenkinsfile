pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        FRONTEND_REPO = 'mern-frontend'
        HELLO_REPO = 'mern-hello-service'
        PROFILE_REPO = 'mern-profile-service'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Get AWS Account ID') {
            steps {
                script {
                    env.AWS_ACCOUNT_ID = sh(
                        script: 'aws sts get-caller-identity --query Account --output text',
                        returnStdout: true
                    ).trim()

                    env.ECR_REGISTRY =
                        "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com"

                    echo "AWS Account ID detected"
                    echo "ECR Registry: ${env.ECR_REGISTRY}"
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password \
                    --region $AWS_REGION | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    $ECR_REGISTRY
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                    docker build \
                    -t $FRONTEND_REPO:latest \
                    ./frontend
                '''
            }
        }

        stage('Build Hello Service Image') {
            steps {
                sh '''
                    docker build \
                    -t $HELLO_REPO:latest \
                    ./backend/helloService
                '''
            }
        }

        stage('Build Profile Service Image') {
            steps {
                sh '''
                    docker build \
                    -t $PROFILE_REPO:latest \
                    ./backend/profileService
                '''
            }
        }

        stage('Tag Images') {
            steps {
                sh '''
                    docker tag \
                    $FRONTEND_REPO:latest \
                    $ECR_REGISTRY/$FRONTEND_REPO:latest

                    docker tag \
                    $HELLO_REPO:latest \
                    $ECR_REGISTRY/$HELLO_REPO:latest

                    docker tag \
                    $PROFILE_REPO:latest \
                    $ECR_REGISTRY/$PROFILE_REPO:latest
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    docker push \
                    $ECR_REGISTRY/$FRONTEND_REPO:latest

                    docker push \
                    $ECR_REGISTRY/$HELLO_REPO:latest

                    docker push \
                    $ECR_REGISTRY/$PROFILE_REPO:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'CI Pipeline completed successfully!'
        }

        failure {
            echo 'CI Pipeline failed. Check the Jenkins console output.'
        }
    }
}