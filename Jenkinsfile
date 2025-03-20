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
          test -f ./build/index.htmls
          ls -la
        '''
      }
    }
  }
}
