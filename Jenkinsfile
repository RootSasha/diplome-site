pipeline { 
    agent any

    environment {
        DOCKER_HUB_REPO = 'sasha22mk/test-site'
        CONTAINER_NAME_SQL = 'sql111'
        CONTAINER_NAME_BEK = 'bek'
        CONTAINER_NAME_FRONT = 'front'
        BEK_PORT = '5034'
    }

    stages {
        stage('Create Docker Network') {
            steps {
                sh 'docker network create baza || true' // Пропустити помилку, якщо мережа вже існує
            }
        }
        stage('Run SQL Server') {
            steps {
                script {
                    // Перевірка чи існує контейнер, і якщо так, то його видалення
                    sh "docker ps -a --filter 'name=${CONTAINER_NAME_SQL}' --format '{{.Names}}' | grep -q ${CONTAINER_NAME_SQL} && docker rm -f ${CONTAINER_NAME_SQL} || true"
                    sh 'docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Qwerty-1" -p 1433:1433 --name sql111 --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest'
                }
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
        stage('Run bek container') {
            steps {
                script {
                    // Перевірка чи порт зайнятий
                    sh "docker ps -a --filter name=${CONTAINER_NAME_BEK} --filter publish=${BEK_PORT} --format '{{.Names}}' | grep -q ${CONTAINER_NAME_BEK} && docker rm -f ${CONTAINER_NAME_BEK} || true"
                    sh "docker run -d -p ${BEK_PORT}:5034 --name bek ${DOCKER_HUB_REPO}:bek"
                }
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
        stage('Run front container') {
            steps {
                script {
                    // Перевірка чи існує контейнер для front
                    sh "docker ps -a --filter name=${CONTAINER_NAME_FRONT} --format '{{.Names}}' | grep -q ${CONTAINER_NAME_FRONT} && docker rm -f ${CONTAINER_NAME_FRONT} || true"
                    sh 'sudo docker run -d -p 81:80 --name front ${DOCKER_HUB_REPO}:front'
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
