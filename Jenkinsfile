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
      steps {
        sh "echo imposible"
      }
    }
  }
}
