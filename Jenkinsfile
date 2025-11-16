pipeline {
    agent any

    parameters {
        choice(
            name: 'TARGET',
            choices: ['front', 'back1', 'back2'],
            description: '배포할 서비스 선택'
        )
    }

    environment {
        AWS_REGION   = 'ap-northeast-1'
        ECR_REGISTRY = '531875446373.dkr.ecr.ap-northeast-1.amazonaws.com'

        ECR_REPO     = "${ECR_REGISTRY}/${TARGET}"      // front / back1 / back2
        CLUSTER      = 'app-cluster'
        SERVICE      = "svc-${TARGET}"                  // svc-front / svc-back1 / svc-back2
        IMAGE_TAG    = "${env.BUILD_NUMBER}"
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
                sh 'chmod +x mvnw || true'
                sh './mvnw clean package -DskipTests'
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'AwsCredentials',
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh """
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set default.region ${AWS_REGION}

                    aws ecr get-login-password --region ${AWS_REGION} \
                      | docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    docker push ${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REPO}:latest
                    """
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'AwsCredentials',
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
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
