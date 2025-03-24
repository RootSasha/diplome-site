pipeline {
    agent any
    environment {
        // Задаємо змінні для імен контейнерів
        SQL_CONTAINER_NAME = 'sql111'
        BEK_CONTAINER_NAME = 'bek'
        FRONT_CONTAINER_NAME = 'front'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Create Docker Network') {
            steps {
                sh 'docker network create baza || true'
            }
        }
        stage('Run SQL Server') {
            steps {
                script {
                    // Перевірка чи контейнер існує і видалення при необхідності
                    sh '''
                        docker ps -a --filter name=${SQL_CONTAINER_NAME} --format {{.Names}} | grep -q ${SQL_CONTAINER_NAME} && docker rm -f ${SQL_CONTAINER_NAME} || true
                        docker run -e ACCEPT_EULA=Y -e MSSQL_SA_PASSWORD=Qwerty-1 -p 1433:1433 --name ${SQL_CONTAINER_NAME} --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest
                    '''
                }
            }
        }
        stage('Commit and Tag SQL Image') {
            steps {
                sh 'docker commit sql111 sasha22mk/test-site:2022-latest'
            }
        }
        stage('Bek copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/monitoring-site/Dockerfile-bek /var/lib/jenkins/workspace/monitoring-site/BackEnd/Amazon-clone/Dockerfile'
            }
        }
        stage('Docker-build-bek') {
            steps {
                sh 'sudo docker build -t sasha22mk/test-site:bek /var/lib/jenkins/workspace/monitoring-site/BackEnd/Amazon-clone/'
            }
        }
        stage('Run bek container') {
            steps {
                script {
                    // Перевірка чи порт вже використовується і видалення контейнера, якщо це так
                    sh '''
                        docker ps -a --filter name=${BEK_CONTAINER_NAME} --format {{.Names}} | grep -q ${BEK_CONTAINER_NAME} && docker rm -f ${BEK_CONTAINER_NAME} || true
                        docker rm -f bek
                        docker run -d -p 5034:5034 --name ${BEK_CONTAINER_NAME} sasha22mk/test-site:bek
                    '''
                }
            }
        }
        stage('Front copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/monitoring-site/Dockerfile-front /var/lib/jenkins/workspace/monitoring-site/FrontEnd/Amazon-clone/Dockerfile'
            }
        }
        stage('Docker-build-front') {
            steps {
                sh 'docker build -t sasha22mk/test-site:front /var/lib/jenkins/workspace/monitoring-site/FrontEnd/Amazon-clone/'
            }
        }
        stage('Run front container') {
            steps {
                script {
                    // Перевірка чи порт вже використовується і видалення контейнера, якщо це так
                    sh '''
                        docker ps -a --filter name=${FRONT_CONTAINER_NAME} --format {{.Names}} | grep -q ${FRONT_CONTAINER_NAME} && docker rm -f ${FRONT_CONTAINER_NAME} || true
                        if docker ps --filter "publish=81" --format {{.Names}} | grep -q front; then
                            echo "Container using port 81 found, removing..."
                            docker rm -f front
                        fi
                        docker run -d -p 81:80 --name ${FRONT_CONTAINER_NAME} sasha22mk/test-site:front
                    '''
                }
            }
        }
        stage('Push SQL Image to Docker Hub') {
            steps {
                sh 'docker push sasha22mk/test-site:2022-latest'
            }
        }
        stage('Push bek to Docker Hub') {
            steps {
                sh 'docker push sasha22mk/test-site:bek'
            }
        }
        stage('Push front to Docker Hub') {
            steps {
                sh 'docker push sasha22mk/test-site:front'
            }
        }
    }
}
