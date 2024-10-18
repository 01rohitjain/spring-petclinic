def IMG_TAG = ''
pipeline {
    agent any
    environment {
            AWS_REGION = 'ap-south-1'      
            ECR_REPO = '730335244747.dkr.ecr.ap-south-1.amazonaws.com'  
            IMAGE_TAG = '' 
        }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/01rohitjain/spring-petclinic.git'
            }
        }
    
        stage('Build using maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build') {
            steps {
                script{
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS_CRED', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    IMG_TAG = "${commitHash}-${env.BUILD_NUMBER}"
                    withEnv(["IMAGE_TAG=$IMG_TAG"]){
                       sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region $AWS_REGION
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                        sudo docker build -t $ECR_REPO/spring-petclinic:$IMAGE_TAG .
                       '''
                    }
                }
                }
            }
        }
        stage('Push to ACR') {
            steps {
                script{
                  withEnv(["IMAGE_TAG=$IMG_TAG"]){
                    sh 'sudo docker push $ECR_REPO/spring-petclinic:$IMAGE_TAG'
                  }
                }
            }
        }
        stage('Display Build Info') {
            steps {
                script{
                    withEnv(["IMAGE_TAG=$IMG_TAG"]){
                        def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                       // Calculate total build time
                        def buildDuration = currentBuild.durationString
                        echo "Total Build Time: ${buildDuration}"
            
                        echo "Build Information:"
                        echo "Build Number: ${env.BUILD_NUMBER}"
                        echo "Commit Hash: ${commitHash}"
                        echo "Docker Image Tag: ${IMAGE_TAG}"
                    }
                }
            }
        }
    }
}
