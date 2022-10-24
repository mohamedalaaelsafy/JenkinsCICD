pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    sh 'composer install'
                        sh 'cp .env.example .env'
                        sh 'php artisan key:generate'
                }
            }

            post {
                success {
                    slackSend (channel: 'jenkins-pipeline', color: '#00FF00', message: "BUILD STAGE SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")

                }
                failure {
slackSend (channel: 'jenkins-pipeline', color: '#FF0000', message: "BUILD STAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
            }
        }
        stage('Unit test') {
            steps {
                sh 'php artisan test'
            }
            post {
                success {
                    slackSend (channel: 'jenkins-pipeline', color: '#00FF00', message: "TEST STAGE SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
                failure {
slackSend (channel: 'jenkins-pipeline', color: '#FF0000', message: "TEST STAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
            }
        }
        stage('Build Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker build -t prod:1.0 .'
                } else if (env.BRANCH_NAME == 'stage') {
                        sh 'docker build -t stage:1.0 .'
                } else {
                        sh 'docker build -t test:1.0 .'
                    }
                }
            }
            post {
                success {
                    slackSend (channel: 'jenkins-pipeline', color: '#00FF00', message: "BUILD IMAGE STAGE SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
                failure {
slackSend (channel: 'jenkins-pipeline', color: '#FF0000', message: "BUILD IMAGE STAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
            }
        }
        stage('Push Image') {
            environment {
                DOCKER_HUB = credentials('DockerHub')
            }
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'echo $DOCKER_HUB_PSW | docker login -u $DOCKER_HUB_USR --password-stdin'
                        sh 'docker tag prod:1.0 mohamedalaaelsafy/prod:1.0'
                        sh 'docker push mohamedalaaelsafy/prod:1.0'
                        sh 'docker rmi mohamedalaaelsafy/prod:1.0'
                        sh 'docker image prune -f'
                        sh 'docker logout'
                } else if (env.BRANCH_NAME == 'stage') {
                        sh 'echo $DOCKER_HUB_PSW | docker login -u $DOCKER_HUB_USR --password-stdin'
                        sh 'docker tag stage:1.0 mohamedalaaelsafy/stage:1.0'
                        sh 'docker push mohamedalaaelsafy/stage:1.0'
                        sh 'docker rmi mohamedalaaelsafy/stage:1.0'
                        sh 'docker image prune -f'
                        sh 'docker logout'
                } else {
                        sh 'echo $DOCKER_HUB_PSW | docker login -u $DOCKER_HUB_USR --password-stdin'
                        sh 'docker tag test:1.0 mohamedalaaelsafy/test:1.0'
                        sh 'docker push mohamedalaaelsafy/test:1.0'
                        sh 'docker rmi mohamedalaaelsafy/test:1.0'
                        sh 'docker image prune -f'
                        sh 'docker logout'
                    }
                }
            }
            post {
                success {
                    slackSend (channel: 'jenkins-pipeline', color: '#00FF00', message: "PUSH IMAGE STAGE SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
                failure {
slackSend (channel: 'jenkins-pipeline', color: '#FF0000', message: "PUSH IMAGE STAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
            }
        }
        stage('Deployment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sshagent(['Stage_Server']) {
                            sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@54.175.231.103 \
                            ./Projects/project_1/main/Deployment/Start_Image.sh
                            '''
                        }
                } else if (env.BRANCH_NAME == 'stage') {
                        sshagent(['Stage_Server']) {
                            sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@54.175.231.103 \
                            ./Projects/project_1/stage/Deployment/Start_Image.sh
                            '''
                        }
                } else {
                        sshagent(credentials : ['Stage_Server']) {
                            sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@54.175.231.103 \
                            ./Projects/project_1/test/Deployment/Start_Image.sh
                            '''
                        }
                    }
                }
            }
            post {
                    success {
                    slackSend (channel: 'jenkins-pipeline', color: '#00FF00', message: "DEPLOYMENT STAGE SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                    }
                    failure {
slackSend (channel: 'jenkins-pipeline', color: '#FF0000', message: "DEPLOYMENT STAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                    }
            }
        }
        stage('Done') {
            steps {
                echo 'DONE...'
            }
            post {
                success {
                    slackSend (channel: 'jenkins-pipeline', color: '#00FF00', message: "CONGRATULATIONS ALL STAGES SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
                failure {
 slackSend (channel: 'jenkins-pipeline', color: '#FF0000', message: "SORRY BUILD FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
                }
            }
        }
    }
}
