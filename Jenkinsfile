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

    stage('Echo') {
      when {
        expression { params.REQUESTED_ACTION == 'test'}
      }
      steps {
        sh "echo imposible"
      }
    }
  }
}
