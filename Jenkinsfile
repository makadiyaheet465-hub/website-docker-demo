pipeline {  
    agent any  
  
    environment {  
        AWS_REGION = 'us-east-1'  
        ECR_REPO = 'website-docker-demo'  
        AWS_ACCOUNT_ID = '4934-9957-9318'  
        IMAGE_TAG = "${env.BUILD_NUMBER}"  
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"  
        LATEST_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest"  
        DEPLOY_SERVER = '3.231.205.55'  
    }  
  
    stages {  
        stage('Checkout') {  
            steps {  
                git branch: 'master', url: 'https://github.com/makadiyaheet465-hub/website-docker-demo.git'  
            }  
        }  
  
        stage('Build Docker Image') {  
            steps {  
                sh 'docker build -t website-docker-demo .'  
            }  
        }  
  
        stage('Tag Docker Image') {  
            steps {  
                sh '''  
                    docker tag website-docker-demo:latest $IMAGE_URI  
                    docker tag website-docker-demo:latest $LATEST_URI  
                '''  
            }  
        }  
  
        stage('Login to ECR') {  
            steps {  
                sh '''  
                    aws ecr get-login-password --region $AWS_REGION |  docker login --username AWS --password-stdin  $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com  
                '''  
            }  
        }  
  
        stage('Push Image to ECR') {  
            steps {  
                sh '''  
                    docker push $IMAGE_URI  
                    docker push $LATEST_URI  
                '''  
            }  
        }  
  
        stage('Deploy to EC2') {  
            steps {  
                sshagent(['e3ee7e85-24e6-4382-9019-1998af17f86e']) {  
                    sh '''  
                        ssh -o StrictHostKeyChecking=no ec2-user@$DEPLOY_SERVER "  
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com &&  
                        docker pull $LATEST_URI &&  
                        docker stop website-demo || true &&  
                        docker rm website-demo || true &&  
                        docker run -d --name website-demo -p 80:80 $LATEST_URI  
                        "  
                    '''  
                }  
            }  
        }  
    }  
  
    post {  
        success {  
            echo 'Website deployed successfully'  
        }  
        failure {  
            echo 'Pipeline failed'  
        }  
    }  
}
