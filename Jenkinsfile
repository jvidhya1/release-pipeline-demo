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
}pipeline {
  agent any
  parameters {
    string(name: 'VERSION', defaultValue: '1.0', description: 'Release version')
    string(name: 'EXPECTED_TEXT', defaultValue: 'Hello', description: 'Text that production verification looks for')
  }
  environment {
    IMAGE = 'release-demo'
  }
  stages {
    stage('Checkout') {
      steps {
        deleteDir()
        checkout scm
      }
    }
    stage('Build Image') {
      steps {
        sh 'echo "Hello from my pipeline - version ${VERSION}" > index.html'
        sh 'docker build -t ${IMAGE}:${VERSION} .'
      }
    }
    stage('Mock Run') {
      steps {
        sh 'docker rm -f release-demo-test || true'
        sh 'docker run -d --name release-demo-test -p 8082:80 ${IMAGE}:${VERSION}'
        sh 'sleep 3'
        sh 'curl -fs http://host.docker.internal:8082 | grep -q "Hello"'
        echo 'Mock run passed'
      }
      post {
        always {
          sh 'docker rm -f release-demo-test || true'
        }
      }
    }
    stage('Approve Production') {
      steps {
        input message: "Approve release ${params.VERSION} to Production?", ok: 'Deploy'
      }
    }
    stage('Deploy Production') {
      steps {
        script {
          env.PREVIOUS_IMAGE = sh(script: 'docker inspect --format "{{.Config.Image}}" release-demo-prod 2>/dev/null || true', returnStdout: true).trim()
          echo "Previous production image: ${env.PREVIOUS_IMAGE}"
        }
        sh 'docker rm -f release-demo-prod || true'
        sh 'docker run -d --name release-demo-prod -p 8081:80 ${IMAGE}:${VERSION}'
      }
    }
    stage('Production Verification') {
      steps {
        script {
          def expected = params.EXPECTED_TEXT ?: 'Hello'
          sleep 3
          def rc = sh(script: "curl -fs http://host.docker.internal:8081 | grep -q '${expected}'", returnStatus: true)
          if (rc != 0) {
            echo 'Verification failed - starting rollback'
            sh 'docker rm -f release-demo-prod || true'
            if (env.PREVIOUS_IMAGE?.trim()) {
              sh "docker run -d --name release-demo-prod -p 8081:80 ${env.PREVIOUS_IMAGE}"
              echo "Rolled back to ${env.PREVIOUS_IMAGE}"
            } else {
              echo 'No previous version to roll back to'
            }
            error("Production verification failed for ${params.VERSION}")
          }
          echo 'Production verification passed'
        }
      }
    }
  }
  post {
    success {
      sh 'echo "Release ${VERSION} deployed on $(date)" > release-${VERSION}.txt'
      archiveArtifacts artifacts: 'release-*.txt'
    }
  }
}