pipeline {
    agent any

    environment {
        DEV_IMAGE = "anand20003/app-dev"
        PROD_IMAGE = "anand20003/app-prod"
        TAG = "${BUILD_NUMBER}"

        DEPLOY_SERVER = "13.232.166.206"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {

                    if (env.BRANCH_NAME == "dev") {
                        sh "docker build -t ${DEV_IMAGE}:${TAG} ."
                    }

                    if (env.BRANCH_NAME == "main") {
                        sh "docker build -t ${PROD_IMAGE}:${TAG} ."
                    }

                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                script {

                    if (env.BRANCH_NAME == "dev") {

                        sh """
                        docker push ${DEV_IMAGE}:${TAG}
                        docker tag ${DEV_IMAGE}:${TAG} ${DEV_IMAGE}:latest
                        docker push ${DEV_IMAGE}:latest
                        """
                    }

                    if (env.BRANCH_NAME == "main") {

                        sh """
                        docker push ${PROD_IMAGE}:${TAG}
                        docker tag ${PROD_IMAGE}:${TAG} ${PROD_IMAGE}:latest
                        docker push ${PROD_IMAGE}:latest
                        """
                    }

                }
            }
        }

        stage('Deploy to EC2') {

            when {
                branch 'main'
            }

            steps {

                sshagent(['ec2-deploy-key']) {

                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOY_SERVER} '

                    docker pull anand20003/app-prod:latest

                    docker stop app || true

                    docker rm app || true

                    docker run -d \
                    --name app \
                    -p 80:80 \
                    anand20003/app-prod:latest
                    '
                    """
                }

            }
        }
    }
}