@Library('jaysee-shared-library')_

pipeline {
    agent any

    environment {
        IMAGE_NAME = 'webapp'
        IMAGE_TAG = 'v1'
        DOCKER_PASSWORD = credentials('docker-password')
        DOCKER_USERNAME = 'ulrichsteve'
        HOST_PORT = 8080
        CONTAINER_PORT = 80
        IP_DOCKER = '172.17.0.1'
    }

    stages {

        stage('Build') {
            steps {
                script {
                    sh '''
                        docker build --no-cache -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '''
                        docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                        curl -I http://$IP_DOCKER
                        sleep 5
                        docker stop $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Release') {
            steps {
                script {
                    sh '''
                        docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Review') {
            environment {
                SERVER_IP = '52.47.197.40'
                SERVER_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Déployer sur l'environnement de review ?", ok: 'Yes'
                    }

                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker rm -f $IMAGE_NAME || true"
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker run -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            curl -I http://$SERVER_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }

        stage('Deploy Staging') {
            environment {
                SERVER_IP = '13.39.158.46'
                SERVER_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Déployer sur l'environnement de staging ?", ok: 'Yes'
                    }

                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker rm -f $IMAGE_NAME || true"
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker run -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            curl -I http://$SERVER_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }

        stage('Deploy Prod') {
            environment {
                SERVER_IP = '15.237.93.85'
                SERVER_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Déployer en production ?", ok: 'Yes'
                    }

                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker rm -f $IMAGE_NAME || true"
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            ssh -o StrictHostKeyChecking=no $SERVER_USERNAME@$SERVER_IP "docker run -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            curl -I http://$SERVER_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                slackNotifier currentBuild.result
            }
        }
    }
}
