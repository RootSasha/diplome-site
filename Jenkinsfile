pipeline {
    agent any
    stages {
        stage('Front copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/site/Dockerfile-front /var/lib/jenkins/workspace/site/FrontEnd/my-app/Dockerfile'
            }
        }
        stage('Docker-build-front') {
            steps {
                sh 'sudo docker build -t front /var/lib/jenkins/workspace/site/FrontEnd/my-app/'
            }
        }
        stage('docker run') {
            steps {
                sh 'docker run -d -p 81:80 front'
            }
        }
    }
}


// /var/lib/jenkins/workspace/site
