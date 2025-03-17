pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'sasha22mk'
        DOCKER_HUB_REPO = 'sasha22mk/test-site'
    }

    stages {
        stage('Run SQL Server') {
            steps {
                sh 'docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Qwerty-1" -p 1433:1433 --name sql111 --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest'
            }
        }
        stage('Commit and Tag SQL Image') {
            steps {
                sh '''
                    docker commit sql111 ${DOCKER_HUB_REPO}:2022-latest
                '''
            }
        }
        stage('Bek copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/monitoring-site/Dockerfile-bek /var/lib/jenkins/workspace/monitoring-site/BackEnd/Amazon-clone/Dockerfile'
            }
        }
        stage('Docker-build-bek') {
            steps {
                sh 'sudo docker build -t ${DOCKER_HUB_REPO}:bek /var/lib/jenkins/workspace/monitoring-site/BackEnd/Amazon-clone/'
            }
        }
        stage('docker run bek') {
            steps {
                sh 'sudo docker run -d -p 5034:5034 ${DOCKER_HUB_REPO}:bek'
            }
        }
        stage('Front copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/monitoring-site/Dockerfile-front /var/lib/jenkins/workspace/monitoring-site/FrontEnd/my-app/Dockerfile'
            }
        }
        stage('Docker-build-front') {
            steps {
                sh 'sudo docker build -t ${DOCKER_HUB_REPO}:front /var/lib/jenkins/workspace/monitoring-site/FrontEnd/my-app/'
            }
        }
        stage('docker run front') {
            steps {
                sh 'sudo docker run -d -p 81:80 ${DOCKER_HUB_REPO}:front'
            }
        }
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                }
            }
        }
        stage('Push SQL Image to Docker Hub') {
            steps {
                sh '''
                    docker push ${DOCKER_HUB_REPO}:2022-latest
                '''
            }
        }
        stage('Push bek to Docker Hub') {
            steps {
                sh '''
                    docker push ${DOCKER_HUB_REPO}:bek
                '''
            }
        }
        stage('Push front to Docker Hub') {
            steps {
                sh '''
                    docker push ${DOCKER_HUB_REPO}:front
                '''
            }
        }
    }
}
