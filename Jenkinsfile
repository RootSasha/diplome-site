pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'sasha22mk/test-site'
        SQL_CONTAINER_NAME = 'sql111'
        BEK_CONTAINER_NAME = 'bek'
        FRONT_CONTAINER_NAME = 'front'
        DOCKER_NETWORK_NAME = 'baza'
    }

    stages {
        stage('Cleanup Containers') {
            steps {
                script {
                    sh '''
                        # Отримання ID всіх існуючих контейнерів
                        SQL111_ID=$(sudo docker ps -aqf "name=${SQL_CONTAINER_NAME}")
                        BEK_ID=$(sudo docker ps -aqf "name=${BEK_CONTAINER_NAME}")
                        FRONT_ID=$(sudo docker ps -aqf "name=${FRONT_CONTAINER_NAME}")

                        # Видалення контейнерів за ID
                        if [ -n "$SQL111_ID" ]; then
                            echo "Видаляємо контейнер ${SQL_CONTAINER_NAME} з ID: $SQL111_ID"
                            sudo docker rm -f $SQL111_ID
                        else
                            echo "Контейнер ${SQL_CONTAINER_NAME} не знайдено."
                        fi

                        if [ -n "$BEK_ID" ]; then
                            echo "Видаляємо контейнер ${BEK_CONTAINER_NAME} з ID: $BEK_ID"
                            sudo docker rm -f $BEK_ID
                        else
                            echo "Контейнер ${BEK_CONTAINER_NAME} не знайдено."
                        fi

                        if [ -n "$FRONT_ID" ]; then
                            echo "Видаляємо контейнер ${FRONT_CONTAINER_NAME} з ID: $FRONT_ID"
                            sudo docker rm -f $FRONT_ID
                        else
                            echo "Контейнер ${FRONT_CONTAINER_NAME} не знайдено."
                        fi
                    '''
                }
            }
        }

        stage('Create Docker Network') {
            steps {
                sh 'sudo docker network create baza || true'
            }
        }

        stage('Run SQL Server') {
            steps {
                sh "sudo docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=Qwerty-1' -p 1433:1433 --name ${SQL_CONTAINER_NAME} --hostname sql1 --network ${DOCKER_NETWORK_NAME} -d mcr.microsoft.com/mssql/server:2022-latest"
            }
        }

        stage('Commit and Tag SQL Image') {
            steps {
                sh "sudo docker commit ${SQL_CONTAINER_NAME} ${DOCKER_HUB_REPO}:2022-latest"
            }
        }

        stage('Bek copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/site/Dockerfile-bek /var/lib/jenkins/workspace/site/BackEnd/Amazon-clone/Dockerfile'
            }
        }

        stage('Docker-build-bek') {
            steps {
                sh "sudo docker build -t ${DOCKER_HUB_REPO}:bek /var/lib/jenkins/workspace/site/BackEnd/Amazon-clone/"
            }
        }

        stage('docker run bek') {
            steps {
                sh '''
                    sleep 15
                    sudo docker run -d -p 5034:5034 --name ${BEK_CONTAINER_NAME} --network ${DOCKER_NETWORK_NAME} ${DOCKER_HUB_REPO}:bek
                '''
            }
        }

        stage('Front copy') {
            steps {
                sh 'cp /var/lib/jenkins/workspace/site/Dockerfile-front /var/lib/jenkins/workspace/site/FrontEnd/my-app/Dockerfile'
            }
        }

        stage('Docker-build-front') {
            steps {
                sh "sudo docker build -t ${DOCKER_HUB_REPO}:front /var/lib/jenkins/workspace/site/FrontEnd/my-app/"
            }
        }

        stage('docker run front') {
            steps {
                sh "sudo docker run -d -p 81:80 --name ${FRONT_CONTAINER_NAME} --network ${DOCKER_NETWORK_NAME} ${DOCKER_HUB_REPO}:front"
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
