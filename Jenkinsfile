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
        stage('Install ALB controller') {
            steps {
                sh 'kubectl apply -k github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master'
                sh 'kubectl get crd'
                sh 'helm repo add eks https://aws.github.io/eks-charts'
                sh 'helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=terraform-eks-demo --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller'
            }
        }
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
                    }
                }
            }
        }
    }
}
