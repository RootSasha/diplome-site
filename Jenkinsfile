pipeline {
    agent any

    environment {
        EKS_CLUSTER_NAME = 'my-eks-cluster'
        AWS_REGION = 'eu-north-1'
        KUBE_CONFIG = credentials('kubeconfig-eks') // Jenkins credentials для kubeconfig
    }

    stages {
        stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                        aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_REGION} --alias ${EKS_CLUSTER_NAME}
                        kubectl apply -f deployment.yaml --kubeconfig <(echo "$KUBE_CONFIG")
                    """
                }
            }
        }
    }
}
