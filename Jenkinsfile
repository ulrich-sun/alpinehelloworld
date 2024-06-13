pipeline {
    environment {
        ID_DOCKER = "ulrichnoumsi"
        IMAGE_NAME = "alpine"
        IMAGE_TAG = "13"
        // PORT_EXPOSED = "${PORT_EXPOSED}"
        // STAGING = "${ID_DOCKER}-staging"
        // PRODUCTION = "${ID_DOCKER}-production"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Clean Environment"
                        docker rm -f $IMAGE_NAME || echo "container does not exist"
                        docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''
                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh ' curl http://172.17.0.1:${PORT_EXPOSED} | grep -q "Hello world!"'
                }
            }
        }
        stage('Clean Container') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop $IMAGE_NAME
                        docker rm $IMAGE_NAME
                    '''
                }
            }
        }
        stage('Login and Push Image on docker hub') {
            agent {
                node {
                    label 'master'
                }
            }
            environment {
                DOCKERHUB_PASSWORD = credentials('dockerhub')
            }
            steps {
                script {
                    sh '''
                        echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
                        docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
        stage('Deploy on the server') {
            steps {
                node {
                    sshagent(['SSH-KEY']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no -l cloud_user 13.40.119.63 curl -k http://
                            echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
                            ssh -o StrictHostKeyChecking=no -l cloud_user 13.40.119.63 docker run --name $IMAGE_NAME -d -p $PORT_EXPOSED:5000 -e PORT=5000 ulrichnoumsi/$IMAGE_NAME:$IMAGE_TAG
                        '''
                    }
                }
            }
        }
    }
}
