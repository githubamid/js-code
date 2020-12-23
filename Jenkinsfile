pipeline {
  environment {
    PROJECT = "eco-volt-297312"
    APP_NAME = "jsapp"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "europe-west6-a"
    IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:${env.GIT_COMMIT}"
    JENKINS_CRED = "${PROJECT}"
  }

  agent {
    kubernetes {
      label 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: jnlp
  - name: docker
    image: docker:18.09
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: tools
    image: argoproj/argo-cd-ci-builder
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    stage('Clone repository') {
      steps {
        git branch: 'master', url: 'https://github.com/githubamid/js-code.git'
      }
    }
    stage('Test') {
      steps {
        container('docker') {
          sh """
            ls -al
          """
        }
      }
    }
    stage('Build and push image') {
      steps {
        container('gcloud') {
          sh """
            PYTHONUNBUFFERED=1 gcloud builds submit -t $IMAGE_TAG .
          """
        }
      }
    }
    stage('Deploy Test') {
      steps {
        container('tools') {
          git branch: 'master', url: 'https://github.com/githubamid/js-deploy.git'
          sh "ls -al"
          sh "git config --global user.email 'cd@cd.com'"

          sh "ls -al"
          
          sh "cd test && kustomize edit set image gcr.io/${PROJECT}/${APP_NAME}:${env.GIT_COMMIT}"
          sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
        }
      }
    }
  }
}
