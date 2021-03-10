pipeline {
    agent any
    
    environment {
        nexus = "ca1559425417.ngrok.io"
        registry = "${nexus}/repository/"
        nexusServer = "http://${nexus}"
        registryCredential = "cred"
        prod_s3_bucket_name = "prod-s3-bucket-frontend-todo-app-www.ekstodoapp.tk"
        stage_s3_bucket_name = "stage-s3-bucket-frontend-todo-app-www.ekstodoapp.tk"
        dockerImageBackand = ''
        dockerImageFrontend = ''
        SONARQUBE_LOGN_PROJECT = credentials('sonarqube_login_project')
        NEXUS_LOGIN = credentials('nexus_login')
        NEXUS_PASSWORD = credentials('nexus_password')
    }
  
    parameters {
        choice (
            choices: ['deploy', 'destroy', 'test'],
            description: '',
            name: 'REQUESTED_ACTION'
        )
    }
    stages {
        
        stage('Branch name') {
            steps {
                echo "Pulling... ${GIT_BRANCH}"
            }
        }
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
              git([url: 'https://github.com/BohdanKrasko/to-do-app', branch: 'main', credentialsId: 'to-do-app-github'])
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

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}

def deploy_job(s3_bucket, env) {
    build job: 'down',
        parameters: [
            [ $class: 'StringParameterValue', name: 'REQUESTED_ACTION', value: "${params.REQUESTED_ACTION}" ],
            [ $class: 'StringParameterValue', name: 'GO_IMAGE', value: "${registry}backend:${BUILD_NUMBER}" ],
            [ $class: 'StringParameterValue', name: 'S3_BUCKET_NAME', value: "${s3_bucket}" ],
            [ $class: 'StringParameterValue', name: 'ENV', value: "${env}" ]
        ]
}

def destroy_job(env) {
    build job: 'cleanJob',
        parameters: [
            [ $class: 'StringParameterValue', name: 'ENV', value: "${env}" ]
        ]
}
