pipeline {
    agent any
    environment{
        REPO_NAME="ecr-repo"
        APP_NAME="next-js"
        IMAGE_TAG = "${BUILD_NUMBER}"
        ACCOUNT_ID = "1234567890"
        REGION = "us-east-2"
    }
    stages{
        
        stage('git clone'){
            steps{
            checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/Rajumettu/hello-world-nextjs']]
        ])
        }
        }
        stage('ECR login'){
            steps{
                sh """
                aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com
                """
}
            }
        stage('Build and push to ECR'){
            steps{
           dir("${WORKSPACE}") {
          sh """
            docker build -t ${REPO_NAME}:${IMAGE_TAG} .
            docker tag ${REPO_NAME}:${IMAGE_TAG} ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
            docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
            """
        }
            }
        }
        stage('Deploy'){
            steps{
                dir("${WORKSPACE}") {
                    sh """
                    sed -i 's/image: build_no/image: ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com\\/${REPO_NAME}:${BUILD_NUMBER}/g' nextjs-depl-svc.yml
                    kubectl apply -f nextjs-depl-svc.yml
                    """
                }
                
            }
        }
        stage('Cleanup workspace'){
            steps{
                script{
                    cleanWs()
                }
            }
        }
    }
}