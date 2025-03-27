pipeline {
    agent any

    environment {
        KUBE_CONFIG = credentials('kubeconfig-eks') // Jenkins credentials для kubeconfig
        DEPLOYMENT_FILE = 'deployment.yaml' // Назва вашого об'єднаного YAML файлу
    }

    stages {
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''#!/bin/bash
                        echo "$KUBE_CONFIG" > kubeconfig.yaml
                        kubectl apply -f ${DEPLOYMENT_FILE} --kubeconfig kubeconfig.yaml
                    '''
                }
            }
        }
    }
}
