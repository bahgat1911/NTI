pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')    // Jenkins credential ID for AWS Access Key ID
        AWS_SECRET_ACCESS_KEY = credentials('bahgataws')            // Jenkins credential ID for AWS Secret Access Key
        AWS_DEFAULT_REGION    = 'eu-north-1'                         // Your AWS region
        ECR_REPOSITORY        = 'bahgat/nti'                     // Your ECR repository name
        AWS_ACCOUNT_ID        = '908027402088'               // Replace with your actual AWS Account ID
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Unset DOCKER_HOST') {
            steps {
                sh '''
                # Unset the DOCKER_HOST environment variable if set
                unset DOCKER_HOST
                echo "DOCKER_HOST has been unset."
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
                    string(credentialsId: 'bahgatoc', variable: 'OPENSHIFT_TOKEN')
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
