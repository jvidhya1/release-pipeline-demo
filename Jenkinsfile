pipeline {
  agent any
  parameters {
    string(name: 'VERSION', defaultValue: '1.0', description: 'Release version')
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build') {
      steps {
        echo "Building release ${params.VERSION}"
        sh 'sh hello.sh'
        sh 'echo "Release $VERSION built on $(date)" > release-$VERSION.txt'
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
        input message: "Approve release ${params.VERSION} to Production?", ok: 'Deploy'
      }
    }
    stage('Deploy') {
      steps {
        echo "Deploying release-demo ${params.VERSION}..."
      }
    }
  }
  post {
    success {
      archiveArtifacts artifacts: 'release-*.txt'
    }
    failure {
      echo "Release ${params.VERSION} failed - rollback would run here"
    }
  }
}