pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-2'
        ECR_REPO   = '124355683348.dkr.ecr.us-east-2.amazonaws.com/eks-apps'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        CLUSTER    = 'acit-eks-group-one'
        KUSTOMIZE_DIR = 'k8s/overlays/dev'
    }

    parameters {
        string(name: 'APP_NAME', defaultValue: 'my-cool-app', description: 'Name of the app to deploy')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $ECR_REPO:$IMAGE_TAG ."
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Update Kustomize Image Tag') {
            steps {
                sh "kustomize edit set image $APP_NAME=$ECR_REPO:$IMAGE_TAG"
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig --name $CLUSTER --region $AWS_REGION
                    kubectl apply -k $KUSTOMIZE_DIR
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment of $APP_NAME version $IMAGE_TAG to $CLUSTER succeeded!"
        }
        failure {
            echo "Deployment failed. Check logs for details."
        }
    }
}
