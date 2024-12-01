pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID      = credentials('aws-access-key-id')    // AWS Access Key ID
        AWS_SECRET_ACCESS_KEY  = credentials('bahgataws')            // AWS Secret Access Key
        AWS_DEFAULT_REGION     = 'eu-north-1'                        // AWS region
        ECR_BACKEND_REPO       = 'bahgat/nti-backend'                // Backend ECR repository
        ECR_FRONTEND_REPO      = 'bahgat/nti-frontend'               // Frontend ECR repository
        AWS_ACCOUNT_ID         = '908027402088'                      // AWS Account ID
        OPENSHIFT_TOKEN        = credentials('bahgatoc')             // OpenShift Token
        OPENSHIFT_PROJECT      = 'bahgat-20-dev'                     // OpenShift Project
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('MaramNTI/nti-project/backend') {
                    script {
                        docker.build("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_BACKEND_REPO}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('MaramNTI/nti-project/frontend') {
                    script {
                        docker.build("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_FRONTEND_REPO}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_DEFAULT_REGION | \
                docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
                '''
            }
        }

        stage('Push Backend Docker Image') {
            steps {
                sh '''
                docker tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_BACKEND_REPO}:${BRANCH_NAME}-${BUILD_NUMBER} \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_BACKEND_REPO}:${BRANCH_NAME}-${BUILD_NUMBER}
                docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_BACKEND_REPO}:${BRANCH_NAME}-${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Frontend Docker Image') {
            steps {
                sh '''
                docker tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_FRONTEND_REPO}:${BRANCH_NAME}-${BUILD_NUMBER} \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_FRONTEND_REPO}:${BRANCH_NAME}-${BUILD_NUMBER}
                docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_FRONTEND_REPO}:${BRANCH_NAME}-${BUILD_NUMBER}
                '''
            }
        }

        stage('Login to OpenShift') {
            steps {
                sh '''
                oc login --token=$OPENSHIFT_TOKEN --server=https://api.sandbox-m2.ll9k.p1.openshiftapps.com:6443
                oc project $OPENSHIFT_PROJECT
                '''
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                sh '''
                # Apply OpenShift deployment YAML
                oc apply -f openshift/deployment.yaml

                # Update backend image
                oc set image deployment/backend backend=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$ECR_BACKEND_REPO:$BRANCH_NAME-$BUILD_NUMBER

                # Update frontend image
                oc set image deployment/frontend frontend=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$ECR_FRONTEND_REPO:$BRANCH_NAME-$BUILD_NUMBER
                '''
            }
        }
    }
}
