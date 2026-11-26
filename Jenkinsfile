
pipeline {

    agent none

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKERHUB_USERNAME = 'suhailmc'
        DOCKERHUB_CREDENTIAL_ID = 'dockerhub-credentials'

        GITHUB_REPOSITORY_URL =
            'https://github.com/suhailmc05-arch/company-website.git'

        IMAGE_NAME = 'company-website'
        CONTAINER_NAME = 'company-website'
        HOST_PORT = '9090'
        CONTAINER_PORT = '80'
    }

    stages {

        // PART 1: BUILD AGENT

        stage('Build and Push Docker Image') {

            agent {
                label 'build-server'
            }

            stages {

                stage('Checkout') {
                    steps {
                        git branch: 'main',
                            url: "${GITHUB_REPOSITORY_URL}"
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        sh '''
                            docker build \
                                -t $IMAGE_NAME:$BUILD_NUMBER .
                        '''
                    }
                }

                stage('Docker Hub Login') {
                    steps {
                        withCredentials([
                            usernamePassword(
                                credentialsId: DOCKERHUB_CREDENTIAL_ID,
                                usernameVariable: 'DH_USER',
                                passwordVariable: 'DH_PASS'
                            )
                        ]) {
                            sh '''
                                echo "$DH_PASS" | docker login \
                                    -u "$DH_USER" \
                                    --password-stdin
                            '''
                        }
                    }
                }

                stage('Push Docker Image') {
                    steps {
                        sh '''
                            docker push $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER
                        '''
                    }

                    post {
                        always {
                            sh 'docker logout || true'
                        }
                    }
                }
            }
        }

        // PART 2: DEPLOYMENT AGENT

        stage('Deploy Application') {

            agent {
                label 'deploy-agent'
            }

            stages {

                stage('Pull Docker Image') {
                    steps {
                        sh '''
                            docker pull \
                                $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER
                        '''
                    }
                }

                stage('Stop Old Container') {
                    steps {
                        sh '''
                            docker stop $CONTAINER_NAME || true
                            docker rm $CONTAINER_NAME || true
                        '''
                    }
                }

                stage('Run New Container') {
                    steps {
                        sh '''
                            docker run -d \
                                --name $CONTAINER_NAME \
                                --restart unless-stopped \
                                -p $HOST_PORT:$CONTAINER_PORT \
                                $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER
                        '''
                    }
                }

                stage('Verify Deployment') {
                    steps {
                        sh '''
                            sleep 5

                            docker ps --filter "name=^/${CONTAINER_NAME}$"

                            curl -fsS \
                                http://localhost:$HOST_PORT \
                                -o /dev/null

                            echo "Website deployed successfully!"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo """
            Deployment Successful!

            Image: ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
            Container: ${CONTAINER_NAME}
            Port: ${HOST_PORT}
            """
        }

        failure {
            echo 'Pipeline failed. Check Jenkins Console Output.'
        }
    }
}
