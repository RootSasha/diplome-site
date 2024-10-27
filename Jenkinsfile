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
                sh 'docker build -t front /var/lib/jenkins/workspace/was-clone/FrontEnd/my-app/'
            }
        }
        stage('docker run') {
            steps {
                sh 'docker run -d -p 80:80 front'
            }
        }
    }
}
