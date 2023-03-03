pipeline {

  environment {
    PROJECT = "ordinal-torch-377801"
    APP_NAME = "cartservice"
    FE_SVC_NAME = "${APP_NAME}-cartservice"
    CLUSTER = "cluster-1"
    CLUSTER_ZONE = "us-central1-c"
    IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:${BUILD_NUMBER}"
    JENKINS_CRED = "${PROJECT}"
  }

  agent {
    kubernetes {
      inheritFrom 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  # serviceAccountName: cd-jenkins
  containers:
  - name: gradle-bld
    image: openjdk:latest
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/google.com/cloudsdktool/cloud-sdk
    command:
    - cat
    tty: true
  - name: kubectl
    image: google/cloud-sdk
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    stage('codebuild') {
      steps {
        container('gradle-bld') {
          sh """
             ls -a && pwd 
          """
        }
      }
    }
    
    stage('Build and push image with Container Builder') {
      steps {
        container('gcloud') {
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${IMAGE_TAG} ./src/"
        }
      }
    } 
    stage('Deploy Dev') {
      steps {
        container('kubectl') {
          
          sh "gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project ordinal-torch-377801"
          sh "kubectl apply -f cartservice.yaml"
          sh "kubectl apply -f cartservice.yaml"
          sh "kubectl set image deployments/cartservice server=${IMAGE_TAG}"
        }
      }
    }
  }
}
