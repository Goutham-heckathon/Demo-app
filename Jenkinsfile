pipeline {

    agent any

    environment {

        AWS_REGION = 'ap-south-1'

        ACCOUNT_ID = '123456789012'

        ECR_REPO = 'demo-app'

        IMAGE_TAG = "${BUILD_NUMBER}"

        IMAGE = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {

            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {

            steps {

                sh '''
                docker build -t demo-app:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login ECR') {

            steps {

                sh '''
                aws ecr get-login-password --region ${AWS_REGION} \
                | docker login \
                --username AWS \
                --password-stdin \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Push Image') {

            steps {

                sh '''

                docker tag demo-app:${IMAGE_TAG} ${IMAGE}

                docker push ${IMAGE}

                '''
            }
        }

        stage('Deploy Green') {

            steps {

                sh '''

                kubectl set image deployment/demo-green \
                demo=${IMAGE}

                '''
            }
        }

        stage('Wait For Rollout') {

            steps {

                sh '''

                kubectl rollout status deployment/demo-green

                '''
            }
        }

        stage('Manual Approval') {

            steps {

                input 'Promote GREEN to Production?'
            }
        }

        stage('Switch Traffic') {

            steps {

                sh '''

                kubectl patch service demo-service \
                -p '{"spec":{"selector":{"app":"demo","version":"green"}}}'

                '''
            }
        }
    }
}