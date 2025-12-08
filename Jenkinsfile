@Library('lamsking-library')_

pipeline {

    environment {
        IMAGE_NAME      = 'appweb'
        IMAGE_TAG       = 'v1'
        DOCKER_PASSWORD = credentials('docker-password')
        DOCKER_USERNAME = 'lamsking'
        HOST_PORT       = 80
        CONTAINER_PORT  = 80
        IP_DOCKER       = 'host.docker.internal'
    }

    agent any

    stages {

        /* ---------------------- CHECKOUT ---------------------- */
        stage('Checkout') {
            steps {
                git url: 'https://github.com/lamsking/jenkins-project.git',
                    branch: 'main',
                    credentialsId: 'github-creds'
            }
        }

        /* ---------------------- BUILD ---------------------- */
        stage('Build') {
            steps {
                script {
                    sh '''
                        docker build --no-cache -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        /* ---------------------- TEST ---------------------- */
        stage('Test') {
            steps {
                script {
                    sh '''
                        docker run --rm -dp $HOST_PORT:$CONTAINER_PORT \
                            --name $IMAGE_NAME $IMAGE_NAME:$IMAGE_TAG

                        sleep 5
                        curl -I http://$IP_DOCKER
                        sleep 5

                        docker stop $IMAGE_NAME
                    '''
                }
            }
        }

        /* ---------------------- RELEASE ---------------------- */
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

        /* ---------------------- DEPLOY REVIEW ---------------------- */
        stage('Deploy Review') {
            environment {
                SERVER_IP       = '98.93.63.247'
                SERVER_USERNAME = 'ubuntu'
                DOCKER_CMD      = 'sudo docker'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Déployer sur l'environnement REVIEW ?", ok: 'Yes'
                    }

                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD rm -f $IMAGE_NAME || echo 'All deleted'"

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo 'Image downloaded'"

                            sleep 30

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME \
                                $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"

                            sleep 5
                            curl -I http://$SERVER_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }

        /* ---------------------- DEPLOY STAGING ---------------------- */
        stage('Deploy Staging') {
            environment {
                SERVER_IP       = '3.93.197.9'
                SERVER_USERNAME = 'ubuntu'
                DOCKER_CMD      = 'sudo docker'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Déployer sur l'environnement STAGING ?", ok: 'Yes'
                    }

                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD rm -f $IMAGE_NAME || echo 'All deleted'"

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo 'Image downloaded'"

                            sleep 30

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME \
                                $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"

                            sleep 5
                            curl -I http://$SERVER_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }

        /* ---------------------- DEPLOY PROD ---------------------- */
        stage('Deploy Prod') {
            environment {
                SERVER_IP       = '54.91.41.168'
                SERVER_USERNAME = 'ubuntu'
                DOCKER_CMD      = 'sudo docker'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Déployer sur l'environnement PROD ?", ok: 'Yes'
                    }

                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD rm -f $IMAGE_NAME || echo 'All deleted'"

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo 'Image downloaded'"

                            sleep 30

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP \
                                "$DOCKER_CMD run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME \
                                $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"

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
