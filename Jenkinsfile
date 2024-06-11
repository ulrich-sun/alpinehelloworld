pipeline{
    environment{
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = ""
        PRODUCTION = ""
    }
    agent none
    stages {
        stage (' Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t eazytraining/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
         stage (' Run container based on build images ') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Clean Environment"
                        docker rm -f $IMAGE_NAME || echo "container does not exist"
                        docker run --name $IMAGE_NAME -d -p $PORT_EXPOSED:5000 -e PORT=5000 eazytraining/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''
                }
            }
        }
        stage (' Test image ') {
            agent any
            steps {
                script {
                    sh '''
                        curl http://172.17.0.1:$PORT_EXPOSED | grep -q 'hello world!'
                    '''
                }
            }
        }
        stage (' Clean Container ') {
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
    }
    post {
        always {
         script {
           slackNotifier currentBuild.result
         }
        } 
    }
}
