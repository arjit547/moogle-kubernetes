pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECR_REPO = 'seasiam'
        ECR_REGISTRY = '435770184212.dkr.ecr.us-east-1.amazonaws.com/seasiam'
        IMAGE_TAG = '435770184212.dkr.ecr.us-east-1.amazonaws.com/seasiam:latest'
        KUBECONFIG_ID = 'my-kubeconfig'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ecr-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                     aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 435770184212.dkr.ecr.us-east-1.amazonaws.com
                     docker build -t seasiam .
                     docker tag seasiam:latest 435770184212.dkr.ecr.us-east-1.amazonaws.com/seasiam:latest
                     docker push 435770184212.dkr.ecr.us-east-1.amazonaws.com/seasiam:latest
                    '''
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'my-aws-secret') {
                    withCredentials([file(credentialsId: "${KUBECONFIG_ID}", variable: 'KUBECONFIG')]) {
                        sh "kubectl apply -f SampleApp.yaml"
                        sh "kubectl apply -f Ingress.yaml"
                        sh "aws elbv2 modify-target-group --target-group-arn arn:aws:elasticloadbalancing:us-east-1:435770184212:targetgroup/k8s-game2048-service2-4e574f2e52/26fcb1d6dcc4c47f --health-check-path /health"
                        sh "aws elbv2 modify-listener --listener-arn arn:aws:elasticloadbalancing:us-east-1:435770184212:listener/app/k8s-game2048-ingress2-4347728cc5/d6ea41e97386194f/9104941f86b3f97d --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:435770184212:targetgroup/k8s-game2048-service2-4e574f2e52/26fcb1d6dcc4c47f"
                    }
                }
            }
        }
    }
}
