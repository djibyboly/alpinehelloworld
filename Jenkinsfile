/* import shared library */
@Library('djibyboly-shared-library')
pipeline {
     environment {
       ID_DOCKER = "${ID_DOCKER_PARAMS}"
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       // PORT_EXPOSED = "80" à paraméter dans le job obligatoirement
       APP_NAME = "dboly"
       STG_API_ENDPOINT = "ip10-0-1-3-ckju9sct654gqaevkvr0-1993.direct.docker.labs.eazytraining.fr"        /* Mettre le couple IP:PORT de votre API eazylabs, exemple 100.25.147.76:1993 */
       STG_APP_ENDPOINT = "ip10-0-1-3-ckju9sct654gqaevkvr0-80.direct.docker.labs.eazytraining.fr"        /* Mettre le couple IP:PORT votre application en staging, exemple 100.25.147.76:8000 */
       PROD_API_ENDPOINT = "ip10-0-1-4-ckju9sct654gqaevkvr0-1993.direct.docker.labs.eazytraining.fr"      /* Mettre le couple IP:PORT de votre API eazylabs, 100.25.147.76:1993 */
       PROD_APP_ENDPOINT = "ip10-0-1-4-ckju9sct654gqaevkvr0-80.direct.docker.labs.eazytraining.fr"
       INTERNAL_PORT = "5000"
       EXTERNAL_PORT = "${PORT_EXPOSED}"
       CONTAINER_IMAGE = "${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}"

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
                    docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:${INTERNAL_PORT} -e PORT=${INTERNAL_PORT} ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
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
                    curl http://172.17.0.1:${PORT_EXPOSED} | grep -q "Hello world!"
                '''
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

      stage('Save Artefact') {
          agent any
          steps {
             script {
               sh '''
                 docker save  ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG > /tmp/alpinehelloworld.tar                 
               '''
             }
          }
     }          
          
     stage ('Login and Push Image on docker hub') {
          agent any
        environment {
           DOCKERHUB_PASSWORD  = credentials('dboly')
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
     
     stage('STAGING - Deploy app') {
      agent any
      steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json 
            """
          }
        }
     }



     stage('PRODUCTION - Deploy app') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any

      steps {
          script {
            sh """
               curl -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json' -d '{"your_name":"${APP_NAME}","container_image":"${CONTAINER_IMAGE}", "external_port":"${EXTERNAL_PORT}", "internal_port":"${INTERNAL_PORT}"}'
               """
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

