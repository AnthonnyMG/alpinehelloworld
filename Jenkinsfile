pipeline {
     environment {
       ID_DOCKER = "${ID_DOCKER_PARAMS}"
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       // PORT_EXPOSED = "80" à paraméter dans le job obligatoirement
       APP_NAME = "TraingTuxDevops"
       STG_API_ENDPOINT = "ip10-0-20-4-cljd31gfplnglid81ukg-1993.direct.docker.labs.eazytraining.fr"
       STG_APP_ENDPOINT = "ip10-0-20-4-cljd31gfplnglid81ukg-80.direct.docker.labs.eazytraining.fr"
       PROD_API_ENDPOINT = "ip10-0-20-5-cljd31gfplnglid81ukg-1993.direct.docker.labs.eazytraining.fr"
       PROD_APP_ENDPOINT = "ip10-0-20-5-cljd31gfplnglid81ukg-80.direct.docker.labs.eazytraining.fr"
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
                    curl -v http://172.17.0.1:${PORT_EXPOSED} | grep -q "Hello world!"
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
    
    // ajouter les credentials docker dans lea parametres
     stage ('Login and Push Image on docker hub') {
          agent any
        environment {
           DOCKERHUB_PASSWORD  = credentials('dockerhub-credentials')
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
       when {
              expression { GIT_BRANCH == 'origin/eazyJenkinslab' }
            }
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
              expression { GIT_BRANCH == 'origin/prod' }
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
     
//   post {
//        success {
//          slackSend (color: '#00FF00', message: "ULRICH - SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) - PROD URL => http://${PROD_APP_ENDPOINT} , STAGING URL => http://${STG_APP_ENDPOINT}")
//          }
//       failure {
//             slackSend (color: '#FF0000', message: "ULRICH - FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
//           }   
//     }     
}
