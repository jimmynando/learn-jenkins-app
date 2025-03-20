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
          node --version
          npm --version
          npm install
          npm run build
          ls -la
        '''
      }
    }
  }
}
