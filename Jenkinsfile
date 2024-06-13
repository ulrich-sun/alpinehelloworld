pipeline {
    environment {
        ID_DOCKER = "ulrichnoumsi"
        IMAGE_NAME = "alpine"
        IMAGE_TAG = "13"
        PORT_EXPOSED = "5000" // Assurez-vous que ce port est défini
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
        stage('Run container based on built image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Clean Environment"
                        docker rm -f ${IMAGE_NAME} || echo "container does not exist"
                        docker run --name ${IMAGE_NAME} -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                    '''
                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh 'curl http://172.17.0.1:${PORT_EXPOSED} | grep -q "Hello world!"'
                }
            }
        }
        stage('Clean Container') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop ${IMAGE_NAME}
                        docker rm ${IMAGE_NAME}
                    '''
                }
            }
        }
        stage('Login and Push Image to Docker Hub') {
            agent any
            environment {
                DOCKERHUB_PASSWORD = credentials('dockerhub')
            }
            steps {
                script {
                    sh '''
                        echo $DOCKERHUB_PASSWORD_PSW | docker login -u ${ID_DOCKER} --password-stdin
                        docker push ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage('Deploy on the server') {
            agent any
            environment {
                DOCKERHUB_PASSWORD = credentials('dockerhub')
                SSH_USER = "cloud_user"
                SSH_HOST = "13.40.119.63"
            }
            steps {
                script {
                    sshagent(['SSH-KEY']) {
                        // sh '''
                        //     echo $DOCKERHUB_PASSWORD_PSW | docker login -u ${ID_DOCKER} --password-stdin
                        //     ssh -o StrictHostKeyChecking=no -l ${SSH_USER} ${SSH_HOST} "
                        //         docker pull ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} &&
                        //         docker rm -f ${IMAGE_NAME} || echo 'container does not exist' &&
                        //         docker run --name ${IMAGE_NAME} -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} &&
                        //         sleep 120 &&
                        //         echo 'the container is running'
                        //     "
                        // '''
                        sh " ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} echo salut la teams "
                    }
                }
            }
        }
    }
}
