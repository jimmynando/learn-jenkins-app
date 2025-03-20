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
          ls -la
        '''
      }
    }

    stage("Test") {
      agent {
        docker {
          image 'node:18-alpine'
          args '-u root:root' // needed root access
          reuseNode true
        }
      }

      steps {
        sh '''
          test -f ./build/index.html
          npm run test
        '''
      }
    }

    stage("E2E") {
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.51.1-noble'
          args '-u root:root' // needed root access
          reuseNode true
        }
      }

      steps {
        sh '''
          npm i serve
          node_modules/.bin/serve -s build &
          sleep 10
          npx playwright test
        '''
      }
    }
  }

  post {
    always {
      junit 'test-results/junit.xml'
    }
  }
}
