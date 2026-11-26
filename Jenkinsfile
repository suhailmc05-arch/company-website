// Jenkinsfile - Company Website CI/CD Pipeline

// Helper: run commands on the deployment server using SSH.
def runOnDeployServer(String command) {
    withCredentials([
        sshUserPrivateKey(
            credentialsId: env.DEPLOYMENT_SSH_CREDENTIAL_ID,
            keyFileVariable: 'SSH_KEY',
            usernameVariable: 'SSH_USER'
        )
    ]) {
        sh "ssh -i \$SSH_KEY -o StrictHostKeyChecking=accept-new " +
           "\$SSH_USER@${env.DEPLOYMENT_SERVER_IP} '${command}'"
    }
}

pipeline {

    agent none

    options {
        skipDefaultCheckout(true)
    }

    environment {

        DOCKERHUB_USERNAME = 'suhailmc'

        DOCKERHUB_REPOSITORY = 'company-website'

        DOCKERHUB_CREDENTIAL_ID = 'dockerhub-credentials'

        DEPLOYMENT_SERVER_IP = '172.31.34.107'

        DEPLOYMENT_SSH_CREDENTIAL_ID = 'deployment-server-ssh'

        GITHUB_REPOSITORY_URL = 'https://github.com/Suhailmc05-arch/Company-Website.git'

        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${DOCKERHUB_REPOSITORY}"

        CONTAINER_NAME = 'company-website'
    }

    stages {

        // PART 1: BUILD SERVER

        stage('Build Server Tasks') {

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

                stage('Docker Image Build') {
                    steps {
                        sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER -t $IMAGE_NAME:latest .'
                    }
                }

                stage('Docker Hub Login') {
                    steps {
                        withCredentials([
                            usernamePassword(
                                credentialsId: env.DOCKERHUB_CREDENTIAL_ID,
                                usernameVariable: 'DH_USER',
                                passwordVariable: 'DH_PASS'
                            )
                        ]) {
                            sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                        }
                    }
                }

                stage('Docker Image Push') {
                    steps {
                        sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
                        sh 'docker push $IMAGE_NAME:latest'
                    }

                    post {
                        always {
                            sh 'docker logout'
                        }
                    }
                }
            }
        }

        // PART 2: DEPLOYMENT SERVER

        stage('Deployment Server Tasks') {

            agent {
                label 'built-in'
            }

            stages {

                stage('Connect to Deployment Server') {
                    steps {
                        script {
                            runOnDeployServer(
                                'hostname && docker --version'
                            )
                        }
                    }
                }

                stage('Pull Latest Image') {
                    steps {
                        script {
                            runOnDeployServer(
                                "docker pull ${IMAGE_NAME}:latest"
                            )
                        }
                    }
                }

                stage('Stop Old Container') {
                    steps {
                        script {
                            runOnDeployServer(
                                "docker stop ${CONTAINER_NAME} || true"
                            )

                            runOnDeployServer(
                                "docker rm ${CONTAINER_NAME} || true"
                            )
                        }
                    }
                }

                stage('Run New Container') {
                    steps {
                        script {
                            runOnDeployServer(
                                "docker run -d --name ${CONTAINER_NAME} " +
                                "--restart unless-stopped -p 9090:80 ${IMAGE_NAME}:latest"
                            )
                        }
                    }
                }

                stage('Verify Deployment') {
                    steps {
                        script {
                            runOnDeployServer(
                                "sleep 3 && docker ps --filter name=${CONTAINER_NAME} " +
                                "&& curl -fsS -o /dev/null http://localhost:9090 " +
                                "&& echo Website is running"
                            )
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Done. Open your deployment server public IP on port 9090.'
        }

        failure {
            echo 'Failed. Read the Console Output to find the stage that failed.'
        }
    }
}
