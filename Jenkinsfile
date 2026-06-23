pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME = 'timotheh/landing-page'
        AWS_VM_IP = '51.20.82.121'
    }

    stages {
        stage('Récupération du code') {
            steps {
                git branch: 'main', url: 'https://github.com/Fvrenn/landing-page.git'
            }
        }

        stage('Build image Docker') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Push DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:latest'
            }
        }

        stage('Déploiement AWS') {
            steps {
                sshagent(['aws-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@51.20.82.121 "
                            docker pull timotheh/landing-page:latest &&
                            docker stop landing-page || true &&
                            docker rm landing-page || true &&
                            docker run -d --name landing-page -p 80:80 timotheh/landing-page:latest
                        "
                    '''
                }
            }
        }
    }

    post {
        success { echo 'Déploiement réussi !' }
        failure { echo 'Échec du déploiement.' }
    }
}
