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

    stage("Tests") {
      parallel {
        stage("Unit tests") {
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
              image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
              args '-u root:root' // needed root access
              reuseNode true
            }
          }

          steps {
            sh '''
              npm i serve
              node_modules/.bin/serve -s build &
              sleep 10
              npx playwright test --reporter=html
            '''
          }
        }
      }
    }    
  }

  post {
    always {
      junit 'jest-results/junit.xml'
      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    }
  }
}
