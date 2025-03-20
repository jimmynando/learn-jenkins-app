pipeline {
  agent any

  stages {
    stage('Build') {
      agent {
        docker {
          image 'node:18-alpine'
          args '-u root:root' // needed root access
          reuseNode true
        }
      }
      steps {
        sh '''          
          ls -la
          rm -rf node_modules
          node --version
          npm --version
          npm install
          npm run test
          ls -la
        '''
      }
    }

    stage("Test") {
      steps {
        sh '''
          test -f ./build/index.html
        '''
      }
    }
  }
}
