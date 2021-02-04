pipeline {
  
  agent any
  
  environment {
    registry = "127.0.0.1:8082/repository/krasko"
    registryCredential = "cred"
    dockerImage = ''
  }
  parameters {
    choice (
      choices: ['deploy', 'destroy', 'test'],
      description: '',
      name: 'REQUESTED_ACTION'
    )
  }
  stages {

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
    
    stage('Build frontend image') {
      when {
        expression { params.REQUESTED_ACTION == 'deploy'}
      }
      steps {
        script {
          dir('app/client') {
            dockerOmage = docker.build registry + ":frontend_" + "$BUILD_NUMBER"
          }
        }
      }
    }
        stage('Deploy frontend image') {
      when {
        expression { params.REQUESTED_ACTION == 'deploy'}
      }
      steps {
        script {
          docker.withRegistry( 'http://nexus:8082', registryCredential ) {
          dockerImage.push()
        }
      }
    }
  }
}
}
