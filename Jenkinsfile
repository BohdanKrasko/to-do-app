pipeline {
  agent any
  
  stages {
    stage('to-do-app - Checkout') {
      steps {
        script {
        checkout([$class: 'GitSCM', 
                  branches: [[name: '*/main']], 
                  doGenerateSubmoduleConfigurations: false, 
                  extensions: [], 
                  submoduleCfg: [], 
                  userRemoteConfigs: [[credentialsId: 'to-do-app-github', url: 'https://github.com/BohdanKrasko/to-do-app']]]) 
      }
      }
    }
    
    stage('Echo') {
      steps {
        sh "echo tytyyty"
      }
    }
  }
}
