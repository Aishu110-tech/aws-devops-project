pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '837868235145'
        ECR_REPOSITORY = 'aws-devops-project-app'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
        EKS_CLUSTER = 'aws-devops-project-eks'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
            }
        }

        stage('ECR Login') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-jenkins-creds']
                ]) {
                    bat '''
                    aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %ECR_REGISTRY%
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                bat 'docker push %IMAGE_NAME%:%BUILD_NUMBER%'
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-jenkins-creds']
                ]) {
                    bat '''
                    aws eks update-kubeconfig --region %AWS_REGION% --name %EKS_CLUSTER%

                    kubectl set image deployment/aws-devops-app aws-devops-app=%IMAGE_NAME%:%BUILD_NUMBER% -n devops-app

                    kubectl rollout status deployment/aws-devops-app -n devops-app
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins-creds']
                ]) {
                    bat '''
                    kubectl get pods -n devops-app
                    kubectl get ingress -n devops-app
                    '''
                }
            }
        }
    }
}