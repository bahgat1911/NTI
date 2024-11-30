pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')    // Jenkins credential ID for AWS Access Key ID
        AWS_SECRET_ACCESS_KEY = credentials('bahgataws')            // Jenkins credential ID for AWS Secret Access Key
        AWS_DEFAULT_REGION    = 'us-east-1'                         // Your AWS region
        ECR_REPOSITORY        = 'your-ecr-repo'                     // Your ECR repository name
        AWS_ACCOUNT_ID        = 'your-aws-account-id'               // Replace with your actual AWS Account ID
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Unset DOCKER_HOST') {
            steps {
                script {
                    // Unset the DOCKER_HOST environment variable if it's set
                    env.DOCKER_HOST = null
                }
                sh '''
                echo "Unset DOCKER_HOST to ensure local Docker socket is used."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REPOSITORY}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'bahgataws', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_DEFAULT_REGION | \
                    docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker tag ${ECR_REPOSITORY}:${BRANCH_NAME}-${BUILD_NUMBER} \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/${ECR_REPOSITORY}:${BRANCH_NAME}-${BUILD_NUMBER}
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/${ECR_REPOSITORY}:${BRANCH_NAME}-${BUILD_NUMBER}
                '''
            }
        }

        stage('Login to OpenShift') {
            steps {
                withCredentials([
                    string(credentialsId: 'openshift-token', variable: 'OPENSHIFT_TOKEN')
                ]) {
                    sh '''
                    oc login --token=$OPENSHIFT_TOKEN --server=https://your-openshift-api-server:6443
                    '''
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                sh '''
                oc project your-project

                # Update the image in the deployment
                oc set image deployment/your-deployment your-container=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/${ECR_REPOSITORY}:${BRANCH_NAME}-${BUILD_NUMBER} --record

                # Optionally, you might need to create or apply resources
                # oc apply -f openshift/deployment.yaml
                '''
            }
        }
    }
}
