pipeline {
  agent any

  stages {
    stage('Build') {
      agent {
        docker {
          image 'node:18-alpine'
          args '-u root:root'
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
          ls -la
          npm run build
          ls -la
        '''
      }
    }
  }
}
