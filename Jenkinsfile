
pipeline {
    environment {
        IMAGE_NAME = "html5"
        APP_EXPOSED_PORT = "80"
        IMAGE_TAG = "latest"
        STAGING = "hc-staging"
        PRODUCTION = "hc-prod"
        DOCKERHUB_ID = "hc61751"
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
        APP_NAME = "hc"
        STG_API_ENDPOINT = "100.26.154.3:1993"
        STG_APP_ENDPOINT = "100.26.154.3:8080"
        PROD_API_ENDPOINT = "100.26.154.3:1993"
        PROD_APP_ENDPOINT = "100.26.154.3:8080"
        INTERNAL_PORT = "5000"
        EXTERNAL_PORT = "${PORT_EXPOSED}"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build .'
              }
           }
       }
       stage('Run container based on builded image') {
          agent any
          steps {
            script {
              sh '''
docker ps
docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME}
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
curl http://172.17.0.1 | grep -q "Welcome"
                '''
              }
           }
       }
       stage('Clean container') {
          agent any
          steps {
             script {
               sh '''
docker stop ${IMAGE_NAME}
docker rm  ${IMAGE_NAME}
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
           expression { GIT_BRANCH == 'origin/master' }
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
  /* post {
    success {
        slackSend (color: '#00FF00', message: "HC HTML5 - SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
    failure {
        slackSend (color: '#FF0000', message: "HC HTML5 - FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }  
  } */
}