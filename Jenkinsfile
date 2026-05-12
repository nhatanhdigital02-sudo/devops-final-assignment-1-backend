pipeline {
    agent any
    environment {
        FULL_IMAGE = "316502579592.dkr.ecr.ap-southeast-1.amazonaws.com/devops-final-assignment-backend:latest"
        TASK_FAMILY = "backend-task-definition"
        CLUSTER_NAME = "final-assignment-ecs-cluster"
        SERVICE_NAME = "final-assignment-be"
    }
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t final-assignment-backend:latest .'
            }
        }
        
        stage('Upload image to ECR') {
            steps {
                // Bắt đầu dùng credential bạn đã tạo ở đây
                withCredentials([usernamePassword(credentialsId: 'aws-creds', 
                                 passwordVariable: 'AWS_SECRET_ACCESS_KEY', 
                                 usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    
                    sh 'aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 316502579592.dkr.ecr.ap-southeast-1.amazonaws.com'
                    sh 'docker tag final-assignment-backend:latest ${FULL_IMAGE}'
                    sh 'docker push ${FULL_IMAGE}'
                }
            }
        }
        
        stage('Update task definition and force deploy ecs service') {
            steps {
                // Tiếp tục dùng credential để có quyền sửa Task Definition trên AWS
                withCredentials([usernamePassword(credentialsId: 'aws-creds', 
                                 passwordVariable: 'AWS_SECRET_ACCESS_KEY', 
                                 usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition ${TASK_FAMILY} --region "ap-southeast-1")
                        NEW_TASK_DEFINITION=$(echo $TASK_DEFINITION | jq --arg IMAGE "${FULL_IMAGE}" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) |  del(.registeredAt)  | del(.registeredBy)')
                        NEW_TASK_INFO=$(aws ecs register-task-definition --region "ap-southeast-1" --cli-input-json "$NEW_TASK_DEFINITION")
                        NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --task-definition ${TASK_FAMILY}:${NEW_REVISION} --force-new-deployment
                    '''
                }
            }
        }
    }
}