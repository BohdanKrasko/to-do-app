pipeline {
  
  agent any
  
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
  }
}
