@Library('lamsking-library')_

pipeline {

    environment {
        IMAGE_NAME       = 'appweb'
        IMAGE_TAG        = 'v1'
        DOCKER_PASSWORD  = credentials('docker-password')
        DOCKER_USERNAME  = 'lamsking'
        HOST_PORT        = 80
        CONTAINER_PORT   = 80
        IP_DOCKER        = 'host.docker.internal'
    }

    agent any

    stages {

        // --- Checkout stage to avoid "fatal: not in a git directory"
        stage('Checkout') {
            steps {
                git url: 'https://github.com/lamsking/jenkins-project.git', branch: 'main', credentialsId: 'github-creds'
            }
        }

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
                SERVER_IP       = '54.162.185.217'
                SERVER_USERNAME = 'ubuntu'
                DOCKER_CMD      = 'sudo docker' // Utiliser sudo si nécessaire
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement de review ?", ok: 'Yes'
                    }

                    sshagent(['key-pair']) {
                        sh """
                            # Connexion Docker sur Jenkins
                            echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin

                            # Supprimer l'ancien conteneur (ignore si inexistant)
                            ssh -o StrictHostKeyChecking=no -l \$SERVER_USERNAME \$SERVER_IP "\$DOCKER_CMD rm -f \$IMAGE_NAME || echo 'All deleted'"

                            # Télécharger la nouvelle image
                            ssh -o StrictHostKeyChecking=no -l \$SERVER_USERNAME \$SERVER_IP "\$DOCKER_CMD pull \$DOCKER_USERNAME/\$IMAGE_NAME:\$IMAGE_TAG || echo 'Image Download successfully'"

                            sleep 10

                            # Lancer le conteneur
                            ssh -o StrictHostKeyChecking=no -l \$SERVER_USERNAME \$SERVER_IP "\$DOCKER_CMD run --rm -dp \$HOST_PORT:\$CONTAINER_PORT --name \$IMAGE_NAME \$DOCKER_USERNAME/\$IMAGE_NAME:\$IMAGE_TAG"

                            sleep 5

                            # Tester l'accès
                            curl -I http://\$SERVER_IP:\$HOST_PORT
                        """
                    }
                }
            }
        }

        stage('Deploy Prod') {
            environment {
                SERVER_IP       = '34.224.101.165'
                SERVER_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement de production ?", ok: 'Yes'
                    }

                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker rm -f $IMAGE_NAME || echo 'All deleted'"
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo 'Image Download successfully'"

                            sleep 30

                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"

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
