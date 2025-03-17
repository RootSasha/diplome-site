pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'your_dockerhub_username'
        DOCKER_HUB_REPO_BEK = 'your_dockerhub_username/bek'
        DOCKER_HUB_REPO_FRONT = 'your_dockerhub_username/front'
        DOCKER_HUB_REPO_DB = 'your_dockerhub_username/sqlserver'
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
                    docker commit sql111 ${DOCKER_HUB_REPO_DB}:latest
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
                sh 'sudo docker build -t bek /var/lib/jenkins/workspace/monitoring-site/BackEnd/Amazon-clone/'
            }
        }
        stage('docker run bek') {
            steps {
                sh 'sudo docker run -d -p 5034:5034 bek'
            }
        }
        stage('Front copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/monitoring-site/Dockerfile-front /var/lib/jenkins/workspace/monitoring-site/FrontEnd/my-app/Dockerfile'
            }
        }
        stage('Docker-build-front') {
            steps {
                sh 'sudo docker build -t front /var/lib/jenkins/workspace/monitoring-site/FrontEnd/my-app/'
            }
        }
        stage('docker run front') {
            steps {
                sh 'sudo docker run -d -p 81:80 front'
            }
        }
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                }
            }
        }
        stage('Tag and Push bek to Docker Hub') {
            steps {
                sh '''
                    docker tag bek ${DOCKER_HUB_REPO_BEK}:latest
                    docker push ${DOCKER_HUB_REPO_BEK}:latest
                '''
            }
        }
        stage('Tag and Push front to Docker Hub') {
            steps {
                sh '''
                    docker tag front ${DOCKER_HUB_REPO_FRONT}:latest
                    docker push ${DOCKER_HUB_REPO_FRONT}:latest
                '''
            }
        }
        stage('Push SQL Image to Docker Hub') {
            steps {
                sh '''
                    docker push ${DOCKER_HUB_REPO_DB}:latest
                '''
            }
        }
    }
}

