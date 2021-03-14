@Library("my-shared-library")_

pipeline {
    agent any
    
    environment {
        nexus = "13aa660b5812.ngrok.io"
        registry = "${nexus}/repository/"
        nexusServer = "http://${nexus}"
        registryCredential = "nexus_cred"
        prod_s3_bucket_name = "prod-s3-bucket-frontend-todo-app-www.ekstodoapp.tk"
        stage_s3_bucket_name = "stage-s3-bucket-frontend-todo-app-www.ekstodoapp.tk"
        dockerImageBackand = ''
        dockerImageFrontend = ''
    }
  
    parameters {
        choice (
            choices: ['deploy', 'destroy', 'test'],
            description: '',
            name: 'REQUESTED_ACTION'
        )
    }
    stages {
    
        stage('Sent notification to Slack') {
            steps {
                script {
                    notifyBuild('STARTED')
                }
            }
        }
        
        stage('Clean workspace') {
            when {
              expression { params.REQUESTED_ACTION == 'deploy'}
            }
            steps {
              cleanWs()
            }
        }
        
        stage('Pull from github') {
            when {
              expression { params.REQUESTED_ACTION == 'deploy'}
            }
            steps {
              git([url: 'git@github.com:BohdanKrasko/to-do-app.git', branch: 'main', credentialsId: 'github_cred'])
            }
        }
        
        stage('Deploy backend image') {
          when {
            expression { params.REQUESTED_ACTION == 'deploy'}
          }
          steps {
            script {
              
              dir('app/go-server') {
                dockerImageBackand = docker.build registry + "backend:" + "$BUILD_NUMBER"
              }
              
              docker.withRegistry( nexusServer, registryCredential ) {
                dockerImageBackand.push()
              }
            }
          }
        }
      
       stage('Deploy') {
            when {
                expression { params.REQUESTED_ACTION == 'deploy'}
            }
            steps {
                script {
                    if ("${GIT_BRANCH}" == "main") {
                        deploy_job("${prod_s3_bucket_name}", 'prod')
                    } else {
                        deploy_job("${stage_s3_bucket_name}", 'stage')
                    }
                }
            }
        }
        
        stage('Destroy') {
            when {
               expression { params.REQUESTED_ACTION == 'destroy'}
            }
            steps {
                script {
                    def userInput = input(
                        id: 'userInput', message: "Destroy enviroment", parameters: [
                        [$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you sure to destroy ']
                    ])

                    if(!userInput) {
                        error "Destroy wasn't confirmed"
                    }
                    
                    if ("${GIT_BRANCH}" == "main") {
                        destroy_job('prod')
                    } else {
                        destroy_job('stage')
                    }
                }
            }
        }   
    }
    
    post {
        always {
            script {
                notifyBuild(currentBuild.result)
            }
        }
       
    } 
}
