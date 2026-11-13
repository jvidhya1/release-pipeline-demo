pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build') {
      steps {
        sh 'sh hello.sh'
        sh 'ls -l'
      }
    }
    stage('Validate') {
      steps {
        sh 'grep -q "Hello" index.html && echo Validation passed'
      }
    }
    stage('Approve Production') {
      steps {
        input message: 'Approve release to Production?', ok: 'Deploy'
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying release-demo...'
      }
    }
  }
  post {
    failure {
      echo 'Build failed - rollback would run here'
    }
  }
}