pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'sasha22mk/test-site'
        SQL_CONTAINER_NAME = 'sql111'
        BEK_CONTAINER_NAME = 'bek'
        FRONT_CONTAINER_NAME = 'front'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm // Отримуємо код з репозиторію
            }
        }
        stage('Cleanup Containers') {
            steps {
                script {
                    sh '''
                        docker ps -a --filter name=${SQL_CONTAINER_NAME} --format {{.Names}} | grep -q ${SQL_CONTAINER_NAME} && docker rm -f ${SQL_CONTAINER_NAME} || true
                        docker ps -a --filter name=${BEK_CONTAINER_NAME} --format {{.Names}} | grep -q ${BEK_CONTAINER_NAME} && docker rm -f ${BEK_CONTAINER_NAME} || true
                        docker ps -a --filter name=${FRONT_CONTAINER_NAME} --format {{.Names}} | grep -q ${FRONT_CONTAINER_NAME} && docker rm -f ${FRONT_CONTAINER_NAME} || true
                    '''
                }
            }
        }
        stage('Create Docker Network') {
            steps {
                sh 'docker network create baza || true' // Створюємо мережу, якщо її немає
            }
        }
        stage('Run SQL Server') {
            steps {
                sh 'docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Qwerty-1" -p 1433:1433 --name ${SQL_CONTAINER_NAME} --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest'
            }
        }
        stage('Commit and Tag SQL Image') {
            steps {
                sh '''
                    docker commit ${SQL_CONTAINER_NAME} ${DOCKER_HUB_REPO}:2022-latest
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
                sh "docker build -t ${DOCKER_HUB_REPO}:bek /var/lib/jenkins/workspace/monitoring-site/BackEnd/Amazon-clone/"
            }
        }
        stage('docker run bek') {
            steps {
                sh '''
                    sleep 15
                    docker run -d -p 5034:5034 --name ${BEK_CONTAINER_NAME} ${DOCKER_HUB_REPO}:bek
                '''
            }
        }
        stage('Front copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/monitoring-site/Dockerfile-front /var/lib/jenkins/workspace/monitoring-site/FrontEnd/my-app/Dockerfile'
            }
        }
        stage('Docker-build-front') {
            steps {
                sh "docker build -t ${DOCKER_HUB_REPO}:front /var/lib/jenkins/workspace/monitoring-site/FrontEnd/my-app/"
            }
        }
        stage('docker run front') {
            steps {
                sh "docker run -d -p 81:80 --name ${FRONT_CONTAINER_NAME} ${DOCKER_HUB_REPO}:front"
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
