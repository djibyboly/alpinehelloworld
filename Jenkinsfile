pipeline {
    environment {
        DOCKERHUB_ID = "${ID_DOCKER_PARAMS}"
        IMAGE_NAME = "alpinehelloword"                    /*alpinehelloworld par exemple*/
        IMAGE_TAG = "latest"                      /*tag docker, par exemple latest*/
        APP_NAME = "dboly"                        /*eazytraining par exemple*/
        STG_API_ENDPOINT = "http://ip10-0-0-3-ckjdmgct654gqaevku80-1993.direct.docker.labs.eazytraining.fr"        /* Mettre le couple IP:PORT de votre API eazylabs, exemple 100.25.147.76:1993 */
        STG_APP_ENDPOINT = "http://ip10-0-0-3-ckjdmgct654gqaevku80-80.direct.docker.labs.eazytraining.fr"        /* Mettre le couple IP:PORT votre application en staging, exemple 100.25.147.76:8000 */
        PROD_API_ENDPOINT = "http://ip10-0-0-4-ckjdmgct654gqaevku80-1993.direct.docker.labs.eazytraining.fr"      /* Mettre le couple IP:PORT de votre API eazylabs, 100.25.147.76:1993 */
        PROD_APP_ENDPOINT = "http://ip10-0-0-4-ckjdmgct654gqaevku80-80.direct.docker.labs.eazytraining.fr"      /* Mettre le couple IP:PORT votre application en production, exemple 100.25.147.76 */
        INTERNAL_PORT = "5000"                                /*"${PARAM_INTERNAL_PORT}"              5000 par défaut*/
        EXTERNAL_PORT = "${PORT_EXPOSED}"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build -t ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG .'
              }
           }
       }
       stage('Run container based on builded image') {
          agent any
          steps {
            script {
              sh '''
                  echo "Cleaning existing container if exist"
                  docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
                  docker run --name $IMAGE_NAME -d -p $APP_EXPOSED_PORT:$INTERNAL_PORT  -e PORT=$INTERNAL_PORT ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                  sleep 5
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                   curl -v 172.17.0.1:$APP_EXPOSED_PORT | grep -q "Hello world!"
                '''
              }
           }
       }
       stage('Clean container') {
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

      stage ('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
                   echo $DOCKERHUB_PASSWORD_PSW | docker login -u $DOCKERHUB_PASSWORD_USR --password-stdin
                   docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }

      stage('STAGING - Deploy app') {
      agent any
      steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}00\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
        }
     
     }
     stage('PROD - Deploy app') {
       when {
           expression { GIT_BRANCH == 'origin/main' }
       }
     agent any

       steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
       }
     }
  }
}

