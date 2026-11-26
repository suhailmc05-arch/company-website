
pipeline {

    agent none

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {

        // Docker Hub configuration
        DOCKERHUB_USERNAME = 'suhailmc'
        DOCKERHUB_CREDENTIAL_ID = 'dockerhub-credentials'

        // GitHub configuration
        GITHUB_REPOSITORY_URL =
            'https://github.com/suhailmc05-arch/company-website.git'

        GIT_BRANCH = 'main'

        // Docker image configuration
        IMAGE_NAME = 'company-website'
        CONTAINER_NAME = 'company-website'

        // Application port configuration
        HOST_PORT = '9090'
        CONTAINER_PORT = '80'
    }

    stages {

        // =========================================
        // PART 1: BUILD SERVER
        // Agent: build-server
        // =========================================

        stage('Build and Push Docker Image') {

            agent {
                label 'build-server'
            }

            stages {

                // Step 1: Checkout source code

                stage('Checkout') {
                    steps {
                        echo 'Checking out source code from GitHub...'

                        git branch: "${GIT_BRANCH}",
                            url: "${GITHUB_REPOSITORY_URL}"
                    }
                }

                // Step 2: Build Docker image

                stage('Build Docker Image') {
                    steps {
                        echo "Building Docker image version ${BUILD_NUMBER}..."

                        sh '''
                            docker build \
                                -t $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER .
                        '''

                        echo 'Docker image built successfully.'
                    }
                }

                // Step 3: Docker Hub authentication

                stage('Docker Hub Login') {
                    steps {
                        echo 'Logging in to Docker Hub...'

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

                // Step 4: Push Docker image

                stage('Push Docker Image') {
                    steps {
                        echo "Pushing image version ${BUILD_NUMBER}..."

                        sh '''
                            docker push \
                                $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER
                        '''

                        echo 'Docker image pushed successfully.'
                    }

                    post {
                        always {
                            sh 'docker logout || true'
                        }
                    }
                }
            }
        }

        // =========================================
        // PART 2: DEPLOYMENT SERVER
        // Agent: deploy-agent
        // =========================================

        stage('Deploy Application') {

            agent {
                label 'deploy-agent'
            }

            stages {

                // Step 5: Pull image from Docker Hub

                stage('Pull Docker Image') {
                    steps {
                        echo "Pulling image version ${BUILD_NUMBER}..."

                        sh '''
                            docker pull \
                                $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER
                        '''
                    }
                }

                // Step 6: Stop existing container

                stage('Stop Old Container') {
                    steps {
                        echo 'Stopping existing container if running...'

                        sh '''
                            docker stop $CONTAINER_NAME || true
                            docker rm $CONTAINER_NAME || true
                        '''
                    }
                }

                // Step 7: Run new container

                stage('Run New Container') {
                    steps {
                        echo 'Starting new application container...'

                        sh '''
                            docker run -d \
                                --name $CONTAINER_NAME \
                                --restart unless-stopped \
                                -p $HOST_PORT:$CONTAINER_PORT \
                                $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER
                        '''
                    }
                }

                // Step 8: Verify deployment

                stage('Verify Deployment') {
                    steps {
                        echo 'Verifying website deployment...'

                        sh '''
                            sleep 5

                            docker ps --filter "name=^/${CONTAINER_NAME}$"

                            curl -fsS \
                                http://localhost:$HOST_PORT \
                                -o /dev/null

                            echo "Website is running successfully!"
                        '''
                    }
                }
            }
        }
    }

    // =========================================
    // PIPELINE POST ACTIONS
    // =========================================

    post {

        success {
            echo """
            =====================================
                 DEPLOYMENT SUCCESSFUL
            =====================================

            Docker Hub:
            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}

            Container:
            ${CONTAINER_NAME}

            Application Port:
            ${HOST_PORT}

            Build Number:
            ${BUILD_NUMBER}

            Website:
            http://<DEPLOYMENT_SERVER_PUBLIC_IP>:${HOST_PORT}

            =====================================
            """
        }

        failure {
            echo '''
            =====================================
                    PIPELINE FAILED
            =====================================

            Check Jenkins Console Output.
            Identify the failed stage and fix it.

            =====================================
            '''
        }

        always {
            echo "Pipeline execution completed. Build: ${BUILD_NUMBER}"
        }
    }
}
