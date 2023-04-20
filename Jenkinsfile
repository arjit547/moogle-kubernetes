pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = 'us-east-2'
        ECR_REPO = 'seasia'
        ECR_REGISTRY = '741979147734.dkr.ecr.us-east-2.amazonaws.com/seasia'
        IMAGE_TAG = '741979147734.dkr.ecr.us-east-2.amazonaws.com/seasia:latest'
        KUBECONFIG_ID = 'my-kubeconfig'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ecr-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                     aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 741979147734.dkr.ecr.us-east-2.amazonaws.com
                     docker build -t seasia .
                     docker tag seasia:latest 741979147734.dkr.ecr.us-east-2.amazonaws.com/seasia:latest
                     docker push 741979147734.dkr.ecr.us-east-2.amazonaws.com/seasia:latest
                    '''
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                withCredentials([kubeconfigFile(credentialsId: "${KUBECONFIG_ID}", variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f SampleApp.yaml"
                    sh "kubectl apply -f Ingress.yaml"
                }
            }
        }
    }
}




