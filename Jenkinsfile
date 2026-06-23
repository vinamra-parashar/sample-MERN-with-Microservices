pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-north-1'
        AWS_ACCOUNT = '928705892222'
        ECR_REGISTRY = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        CLUSTER_NAME = 'sample-mern-microservices'
        NAMESPACE = 'mern-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Source') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vinamra-parashar/sample-MERN-with-Microservices.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t frontend:${IMAGE_TAG} ./frontend
                docker build -t hello-service:${IMAGE_TAG} ./helloService
                docker build -t profile-service:${IMAGE_TAG} ./profileService
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login \
                --username AWS \
                --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag Images') {
            steps {
                sh '''
                docker tag frontend:${IMAGE_TAG} ${ECR_REGISTRY}/frontend:${IMAGE_TAG}
                docker tag hello-service:${IMAGE_TAG} ${ECR_REGISTRY}/hello-service:${IMAGE_TAG}
                docker tag profile-service:${IMAGE_TAG} ${ECR_REGISTRY}/profile-service:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                docker push ${ECR_REGISTRY}/frontend:${IMAGE_TAG}
                docker push ${ECR_REGISTRY}/hello-service:${IMAGE_TAG}
                docker push ${ECR_REGISTRY}/profile-service:${IMAGE_TAG}
                '''
            }
        }

        stage('Configure kubectl') {
            steps {
                sh '''
                aws eks update-kubeconfig \
                --region ${AWS_REGION} \
                --name ${CLUSTER_NAME}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl set image deployment/frontend \
                frontend=${ECR_REGISTRY}/frontend:${IMAGE_TAG} \
                -n ${NAMESPACE}

                kubectl set image deployment/helloService \
                helloService=${ECR_REGISTRY}/hello-service:${IMAGE_TAG} \
                -n ${NAMESPACE}

                kubectl set image deployment/profileService \
                profileService=${ECR_REGISTRY}/profile-service:${IMAGE_TAG} \
                -n ${NAMESPACE}
                '''
            }
        }

        stage('Wait for Rollout') {
            steps {
                sh '''
                kubectl rollout status deployment/frontend -n ${NAMESPACE}
                kubectl rollout status deployment/helloService -n ${NAMESPACE}
                kubectl rollout status deployment/profileService -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "===== Pods ====="
                kubectl get pods -n ${NAMESPACE}

                echo ""
                echo "===== Services ====="
                kubectl get svc -n ${NAMESPACE}

                echo ""
                echo "===== Deployments ====="
                kubectl get deployments -n ${NAMESPACE}
                '''
            }
        }
    }

    post {

        success {
            echo '===================================='
            echo 'Application deployed successfully.'
            echo "Image Tag : ${IMAGE_TAG}"
            echo '===================================='
        }

        failure {
            echo '===================================='
            echo 'Pipeline failed.'
            echo 'Check console logs.'
            echo '===================================='
        }

        always {
            cleanWs()
        }
    }
}