@Library('github.com/releaseworks/jenkinslib') _

pipeline {
  
  agent any
  
  environment {
    //registry = "127.0.0.1:8082/repository/krasko"
    registry = "2879fbb1a708.ngrok.io/repository/krasko"
    nexusServer = "http://2879fbb1a708.ngrok.io"
    registryCredential = "cred"
    dockerImageBackand = ''
    dockerImageFrontend = ''
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
            dockerImageFrontend = docker.build registry + ":frontend_" + "$BUILD_NUMBER"
          }
          
          docker.withRegistry( nexusServer, registryCredential ) {
            dockerImageFrontend.push()
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
            dockerImageBackand = docker.build registry + ":backend_" + "$BUILD_NUMBER"
          }
          
          docker.withRegistry( nexusServer, registryCredential ) {
            dockerImageBackand.push()
          }
        }
      }
    }
    
    stage('Terrafom') {
      when {
        expression { params.REQUESTED_ACTION == 'deploy'}
      }
      steps {
        
        dir('terraform') {
          withAWS(credentials:'aws_cred', region:'eu-west-3') {
            sh 'terraform init'
            sh 'terraform plan'
            sh 'terraform apply -auto-approve'
         }
        }
      }
    }
    
    stage('Deploy todo app in EKS cluster') {
      when {
        expression { params.REQUESTED_ACTION == 'deploy'}
      }
      steps {
        dir('kubernetes') {
          withAWS(credentials:'aws_cred', region:'eu-west-3') {
            withEnv(["KUBECONFIG=/var/jenkins_home/workspace/to-do-app_main/terraform/kubeconfig_my-cluster"]) {
              sh (
                label: 'Run app',
                script: """#!/usr/bin/env bash 
                helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
                helm repo update
                helm install ingress-nginx ingress-nginx/ingress-nginx
                sleep 30
                kubectl create secret docker-registry regcred --docker-server=2879fbb1a708.ngrok.io --docker-username=admin --docker-password=admin123
                kubectl apply -f app/mongo.yml
                echo dfpepfep[kfp[e[f[el[pfoepwop[oo[p
                echo $registry
                helm install go helm/to-do-backend --set imageNamne=$registry + ":backed_" + $BUILD_NUMBER
                helm install react helm/react-to-do --set imageName="$registry + :frontend + $BUILD_NUMBER"
                """
            )
          }
        }
      }
    }
  }
  
    stage('Ansible add A record') {
      when {
        expression { params.REQUESTED_ACTION == 'deploy'}
      }
      steps {
        dir('ansible') {
          withAWS(credentials:'aws_cred', region:'eu-west-3') {
            sh 'ansible-playbook dns.yml'
          }
        }
      }
    }  
//    stage('Add A record') {
//      steps {
//        script {
//        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
//        AWS("--region=eu-west-3 s3  ls")
//      }
//      }
//      }
//      
//    }
  }
}
