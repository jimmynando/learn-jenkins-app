pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID = 'e8d44f88-1e27-4d1d-a91d-a5bdfcffd834'
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    REACT_APP_VERSION = '1.0.$BUILD_ID'
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

    stage("Deploy Staging") {
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
          args '-u root:root' // needed root access
          reuseNode true
        }
      }

      environment {
        CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
      }

      steps {
        sh '''
          npm install netlify-cli -g
          netlify --version
          echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
          netlify status
          netlify deploy --dir=build --json > deploy-output.json
          CI_ENVIRONMENT_URL=$(npx node-jq -r '.deploy_url' deploy-output.json)
          npx playwright test --reporter=html
        '''
      }

      post {
        always {
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
        }
      }
    }

    // stage('Approval') {
    //   steps {
    //     echo 'Pending approval...'
    //     timeout(time: 1, unit: 'MINUTES') {
    //       input message: 'Ready to deploy?', ok: 'Yes, I am sure I want to deploy'
    //     }
    //   }
    // }

    stage("Deploy Prod") {
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
          args '-u root:root' // needed root access
          reuseNode true
        }
      }

      environment {
        CI_ENVIRONMENT_URL = 'https://jimmytri.netlify.app'
      }

      steps {
        sh '''
          npm install netlify-cli -g
          netlify --version
          echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
          netlify status
          netlify deploy --dir=build --prod
          npx playwright test --reporter=html
        '''
      }

      post {
        always {
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
        }
      }
    }
  }  
}
