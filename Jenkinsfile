pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID="210651441624"
        AWS_DEFAULT_REGION="ap-northeast-2"
        IMAGE_REPO_NAME="ecr-for-ecs-tutorial"
        IMAGE_TAG="latest"
        REPOSITORY_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        PROJECT_NAME = 'test-github-pipeline' //Jenkins Project Name
        CLUSTER_NAME = 'cluster-for-ecs-tutorial'
        SERVICE_NAME = 'service-for-ecs-tutorial'
        BRANCH = 'main'
        GIT_URL = 'https://github.com/sinjy1203/ecs-tutorial.git'
    }

    stages {
      
        stage('AWS ECR Login'){
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
            }
        }

        stage ('Git Clone'){ 
        	steps { 
            	git branch: BRANCH, 
                credentialsId: 'GIT_ACCOUNT', 
                url: GIT_URL
            } 
        } 
        

        stage('Gradle Build') {
            steps {
                sh './gradlew build -x test'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage ('ECR Push') {
            steps {
                script {
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}"
                    sh "docker push ${REPOSITORY_URI}"
                }
            }
        }

        stage('Clean docker image') {
            steps{
                sh "docker rmi -f ${IMAGE_REPO_NAME} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
                sh './gradlew clean'
            }
        }

        stage ('ECS Deploy') {
            steps {
                script {
                    sh "aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --force-new-deployment"
                }
            }
        }
        
    }
   
}