pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-northeast-1'
        ECR_REGI  = '531875446373.dkr.ecr.ap-northeast-1.amazonaws.com'
        ECR_REPO  = "${ECR_REGI}/back1"
        CLUSTER   = 'app-cluster'
        SERVICE   = 'svc-back1'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/sdtrd011/final-lab-petclinic.git'
            }
        }

        stage('Build Jar') {
            steps {
                sh """
                docker run --rm \
                  -v \$PWD:/workspace \
                  -w /workspace \
                  maven:3.9.9-eclipse-temurin-25 \
                  mvn -B clean package -DskipTests
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REPO}:latest
                """
            }
        }

        stage('Login & Push to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'AwsCredentials',
                                                 usernameVariable: 'AWS_ACCESS_KEY_ID',
                                                 passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set default.region ${AWS_REGION}

                    aws ecr get-login-password --region ${AWS_REGION} \
                      | docker login --username AWS --password-stdin ${ECR_REPO}

                    docker push ${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REPO}:latest
                    """


                    
             }
            }
        }

        stage('Deploy to ECS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'AwsCredentials',
                                                 usernameVariable: 'AWS_ACCESS_KEY_ID',
                                                 passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set default.region ${AWS_REGION}

                    aws ecs update-service \
                      --cluster ${CLUSTER} \
                      --service ${SERVICE} \
                      --force-new-deployment \
                      --region ${AWS_REGION}
                    """
                }
            }
        }
    }
}
