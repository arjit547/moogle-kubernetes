pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = 'us-east-2'
        ECR_REPO = 'mooglelabs'
        IMAGE_TAG = '741979147734.dkr.ecr.us-east-1.amazonaws.com/mooglelabs:latest'
        KUBECONFIG_ID = 'my-kubeconfig'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 741979147734.dkr.ecr.us-east-1.amazonaws.com"
                    sh "docker build -t mooglelabs ."
                    sh "docker tag mooglelabs:latest 741979147734.dkr.ecr.us-east-1.amazonaws.com/mooglelabs:latest"
                    sh "docker push 741979147734.dkr.ecr.us-east-1.amazonaws.com/mooglelabs:latest"
                }
            }
        
        stage('Deploy to EKS') {
            steps {
                withCredentials([kubeconfigFile(credentialsId: "${KUBECONFIG_ID}", variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f SampleApp.yaml"
                    sh "kubectl apply -f Ingress.yaml "
                }
            }
        }
}
