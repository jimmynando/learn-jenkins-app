pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID = 'e8d44f88-1e27-4d1d-a91d-a5bdfcffd834'
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
  }

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
          npm run build
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

          post {
            always {
              junit 'jest-results/junit.xml'
            }
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

          post {
            always {
              publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            }
          }
        }
      }
    }

    stage('Deploy') {
      agent {
        docker {
          image 'node:18-alpine'
          args '-u root:root' // needed root access
          reuseNode true
        }
      }

      steps {
        sh '''
          npm install netlify-cli -g
          netlify --version
          echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
          netlify status
          netlify deploy --dir=build --prod
        '''
      }
    }
  }  
}
