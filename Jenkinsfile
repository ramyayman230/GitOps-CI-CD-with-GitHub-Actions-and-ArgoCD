pipeline {
  agent any 


  parameters {

    choice(
      name: 'Action',
      choices: ['DEPLOY','ROLLBACK'],
      description: 'please choose'
    )

    string(
      name: 'ROLLBACK_VERSION',
      defaultValue: '',
      description: 'previos docker image tag'
    )
  }
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
            anyOf {
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
          when {
            expression { params.ACTION == 'DEPLOY'}
          }
            steps{
                sh '''
                docker build -t $IMAGE_NAME:$BUILD_NUMBER .               
                '''
            }
        }

        stage ('deploy container'){
          when {
            expression { params.ACTION == 'DEPLOY'}
          }
            steps{
                sh '''
                docker rm -f ${CONTAINER_NAME} || true
                docker run -d --name $CONTAINER_NAME -p $APP_PORT:$APP_PORT $IMAGE_NAME:$BUILD_NUMBER

                '''
            }
        }

      stage ('production approval gate') {
             when {
               expression { params.ACTION == 'DEPLOY'}
             }
             steps {
               timeout ( time: 10 , unit: 'MINUTES'){
                 input(
                   message: 'do you approve deploying this application?',
                   ok: 'Approve Deployment',
                   submitter: 'admin'
                 )
               }
               echo 'Deployment Approved! , Continuing'
             }
      }
      

        stage ('deployment complete'){
          when {
            expression { params.ACTION == 'DEPLOY'}
          }
               steps {

                  sh '''

                  docker ps --filter "name=${CONTAINER_NAME}"
                  echo "Done deployment"

                  '''
                   
               }

               }
         stage ('Rollback'){
           when {
             expression (params.ACTION='ROLLBACK')
           }
           steps {
             sh " " "
             docker rm -f ${CONTAINER_NAME} || true
             docker run -d -ti --name $CONTAINER_NAME -p $APP_PORT:$APP_PORT $IMAGE_NAME:${params.ROLLBACK_VERSION}

             " " "
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

