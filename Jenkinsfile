pipeline {
  
  agent any
  
  environment {
    registry = "127.0.0.1:8082/repository/krasko"
    registryCredential = "cred"
    dockerImage = ''
    SONARQUBE_LOGN_PROJECT = credentials('sonarqube_login_project')
  }
  parameters {
    choice (
      choices: ['deploy', 'destroy', 'test'],
      description: '',
      name: 'REQUESTED_ACTION'
    )
  }
  
  tools {
    terraform 'Terraform'
  }
  stages {

//  stage('Clean workspace') {
//    when {
//      expression { params.REQUESTED_ACTION == 'deploy'}
//    }
//    steps {
//      cleanWs()
//    }
//  }
    
//  stage('Pull from github') {
//    when {
//      expression { params.REQUESTED_ACTION == 'deploy'}
//    }
//    steps {
//      git([url: 'https://github.com/BohdanKrasko/to-do-app', branch: 'main', credentialsId: 'to-do-app-github'])
//    }
//  }  
    stage('Terrafom') {
      when {
        expression { params.REQUESTED_ACTION == 'deploy'}
      }
 //     environment { 
 //       //KUBECONFIG= sh (returnStdout: true, script: 'echo "$(cat /var/jenkins_home/workspace/to-do-app_main/terraform/kubeconfig_my-cluster)"')
 //       foo = sh(
 //         returnStdout: true, 
 //         script: 'cat /var/jenkins_home/workspace/to-do-app_main/terraform/kubeconfig_my-cluster'
 //       )
 //    }

      steps {
        
        dir('terraform') {
          withAWS(credentials:'aws_cred', region:'eu-west-3') {
  //        sh 'terraform init'
  //        sh 'terraform plan'
  //        sh 'terraform apply -auto-approve'
  //          sh (
  //            label: "Kube",
  //            script: """#!/usr/bin/env bash
  //            echo $foo
  //            kubectl get pods
  //            """
            script {
              foo = sh(
                returnStdout: true, 
                script: 'cat /var/jenkins_home/workspace/to-do-app_main/terraform/kubeconfig_my-cluster'
              )
            }
            sh 'echo dsfhkdj'
            sh 'echo $foo'
  //          )
          }
        }
      }
    }

    
  }
}
