pipeline {
    agent any
    stages {
        stage('Front copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/was-clone/Dockerfile-front /var/lib/jenkins/workspace/was-clone/FrontEnd/my-app/Dockerfile'
            }
        }
        stage('Docker-build-front') {
            steps {
                sh 'sudo docker build -t front /var/lib/jenkins/workspace/was-clone/FrontEnd/my-app/'
            }
        }
        stage('docker run') {
            steps {
                sh 'docker run -d -p 81:80 front'
            }
        }
    }
}
