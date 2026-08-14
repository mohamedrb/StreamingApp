pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = 'YOUR_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com'

        FRONTEND_IMAGE = "${ECR_REGISTRY}/streamingapp-frontend"
        BACKEND_IMAGE  = "${ECR_REGISTRY}/streamingapp-backend"

        IMAGE_TAG = "${BUILD_NUMBER}"

        CLUSTER_NAME = 'streamingapp-cluster'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Frontend Test') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm test -- --watchAll=false'
                }
            }
        }

        stage('Backend Test') {
            steps {
                dir('backend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                    docker build \
                    -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                    ./frontend

                    docker build \
                    -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                    ./backend
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password \
                    --region ${AWS_REGION} |
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}

                    docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}

                    docker push ${BACKEND_IMAGE}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                    aws eks update-kubeconfig \
                    --region ${AWS_REGION} \
                    --name ${CLUSTER_NAME}

                    helm upgrade --install streamingapp \
                    ./helm/streamingapp \
                    --namespace streamingapp \
                    --create-namespace \
                    --set frontend.image.repository=${FRONTEND_IMAGE} \
                    --set frontend.image.tag=${IMAGE_TAG} \
                    --set backend.image.repository=${BACKEND_IMAGE} \
                    --set backend.image.tag=${IMAGE_TAG}
                """
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    kubectl get pods -n streamingapp
                    kubectl get services -n streamingapp
                '''
            }
        }
    }
}