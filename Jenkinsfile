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
    

    stage('Deploy frontend image') {
      when {
        expression { params.REQUESTED_ACTION == 'deploy'}
      }
      steps {
        script {
          
          dir('app/client') {
            dockerImage = docker.build registry + ":frontend_" + "$BUILD_NUMBER"
          }
          
          docker.withRegistry( 'http://127.0.0.1:8082', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    
    stage('Deploy backend image') {
      when {
        expression { params.REQUESTED_ACTION == 'deploy'}
      }
      steps {
        script {
          
          dir('app/go-server') {
            dockerImage = docker.build registry + ":backend_" + "$BUILD_NUMBER"
          }
          
          docker.withRegistry( 'http://127.0.0.1:8082', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    
    stage('Sonarqube') {
      environment {
        scannerHome = tool 'SonarQubeScanner'
      }
      steps {
        withSonarQubeEnv('sonarqube') {
            
            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=project -Dsonar.sources=. -Dsonar.host.url=http://sonarqube:9000/ -Dsonar.login=5b8775287636bf81aea706e8eec91979e86f8d01"
             
        }
      }
    }
  }
}
