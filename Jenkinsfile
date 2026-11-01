pipeline {
  agent any 

  environment {
      IMAGE_NAME = 'jenkinfreestyle'
      CONTAINER_NAME = 'testcontainer'
      APP_PORT = '3001'
  } 

    stages {
        stage ('checkout code'){
            steps{
                git branch:'Master',
                    url: 'https://github.com/ramyayman230/GitOps-CI-CD-with-GitHub-Actions-and-ArgoCD.git'
               }           
        }

        stage ('inpsect workspace'){
          when {
            anyof {
              changeset 'Dockerfile'
              changeset  'app.js'
            }
          }
            steps{
                sh '''
                 pwd
                 whoami 
                 git rev-parse --short HEAD
                '''
            }    
        }

        stage ('build image'){
            steps{
                sh '''
                docker build -t $IMAGE_NAME:$BUILD_NUMBER .               
                '''
            }
        }

        stage ('deploy container'){
            steps{
                sh '''
                docker rm -f ${CONTAINER_NAME} || true
                docker run -d --name $CONTAINER_NAME -p $APP_PORT:$APP_PORT $IMAGE_NAME:$BUILD_NUMBER

                '''
            }
        }

        stage ('deployment complete'){
               steps {

                  sh '''

                  docker ps --filter "name=${CONTAINER_NAME}"
                  echo "Done deployment"

                  '''
                   
               }

               } 
        
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}

