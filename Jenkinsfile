pipeline {

    agent any

    environment {

        DOCKER_IMAGE = "suryadevara2025/flask-cicd-app"

        DOCKER_CREDENTIALS = "dockerhub-credentials"

        EC2_HOST = "52.66.238.68"

    }

    stages {

        stage('Checkout') {

            steps {

                echo 'Downloading source code from GitHub'

                checkout scm

            }

        }

        stage('Build') {

            steps {

                echo 'Installing Python dependencies'

                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install -r requirements.txt
                '''

            }

        }

        stage('Test') {

            steps {

                echo 'Running application tests'

                sh '''
                    .venv/bin/python -m pytest -v
                '''

            }

        }

        stage('Docker Build') {

            steps {

                echo 'Building Docker image'

                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                '''

            }

        }

        stage('Docker Push') {

            steps {

                echo 'Pushing image to Docker Hub'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                        docker logout
                    '''

                }

            }

        }

        stage('Deploy') {

            steps {

                echo 'Deploying application to AWS EC2'

                sshagent(credentials: ['ec2-ssh-key']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ubuntu@${EC2_HOST} \
                        "/home/ubuntu/deploy.sh ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    '''

                }

            }

        }

    }

    post {

        success {

            echo 'CI/CD Pipeline completed successfully'

        }

        failure {

            echo 'CI/CD Pipeline failed. Check console output.'

        }

    }

}
