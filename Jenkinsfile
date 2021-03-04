pipeline {
    agent any
    
    environment {
        //registry = "127.0.0.1:8082/repository/krasko"
        registry = "2536f8ab9d87.ngrok.io/repository/"
        nexusServer = "http://2536f8ab9d87.ngrok.io"
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
       
       stage('Deploy Stage') {
            steps {
                script {
                    def releaseJob = build job: 'down',
                    parameters: [
                        [ $class: 'StringParameterValue', name: 'REQUESTED_ACTION', value: "${params.REQUESTED_ACTION}" ],
                        [ $class: 'StringParameterValue', name: 'GO_IMAGE', value: "${registry}backend:${BUILD_NUMBER}" ],
                        [ $class: 'StringParameterValue', name: 'S3_BUCKET_NAME', value: "${stage_s3_bucket_name}" ],
                        [ $class: 'StringParameterValue', name: 'DIR', value: "stage/app" ]
                    ]
                    
                    if (releaseJob.result == "SUCCESS") {
                        echo "SUCCESS downstream job"
                    } else {
                        echo "Error"
                    }
                }
            }
        }
        
        stage('Add fronted to S3 to stage') {
          when {
            expression { params.REQUESTED_ACTION == 'deploy'}
          }
          steps {
              dir('app/client') {
                sh "yarn install"
                sh "REACT_APP_HOST=https://stage.go.ekstodoapp.tk yarn build && aws s3 sync build/ s3://${stage_s3_bucket_name}"
            }
          }
        }
        
        stage('Deploy Prod') {
            steps {
                script {
                    def releaseJob = build job: 'down',
                    parameters: [
                        [ $class: 'StringParameterValue', name: 'REQUESTED_ACTION', value: "${params.REQUESTED_ACTION}" ],
                        [ $class: 'StringParameterValue', name: 'GO_IMAGE', value: "${registry}backend:${BUILD_NUMBER}" ],
                        [ $class: 'StringParameterValue', name: 'S3_BUCKET_NAME', value: "${prod_s3_bucket_name}" ],
                        [ $class: 'StringParameterValue', name: 'DIR', value: "prod/app" ]
                    ]
                    
                    if (releaseJob.result == "SUCCESS") {
                        echo "SUCCESS downstream job"
                    } else {
                        echo "Error"
                    }
                }
            }
        }
        
        stage('Add fronted to S3 to prod') {
          when {
            expression { params.REQUESTED_ACTION == 'deploy'}
          }
          steps {
              dir('app/client') {
                sh "yarn install"
                sh "REACT_APP_HOST=https://prod.go.ekstodoapp.tk yarn build && aws s3 sync build/ s3://${prod_s3_bucket_name}"
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
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

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
