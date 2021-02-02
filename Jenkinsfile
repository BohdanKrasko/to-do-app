pipeline {
  agent any
      
      
        script {
        checkout([$class: 'GitSCM', 
                  branches: [[name: '*/main']], 
                  doGenerateSubmoduleConfigurations: false, 
                  extensions: [], 
                  submoduleCfg: [], 
                  userRemoteConfigs: [[credentialsId: 'to-do-app-github', url: 'https://github.com/BohdanKrasko/to-do-app']]]) 
      }
     
   
  stages {

    stage('Echo') {
      steps {
        sh "echo imposible"
      }
    }
  }
}
