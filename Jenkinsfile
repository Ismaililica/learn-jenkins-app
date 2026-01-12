pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '337f0d62-6d00-4f42-8ec4-907379f12d10'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }
        } 
        stage(' Tests'){
        
        parallel{
                stage('Unit Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    
                    steps{
                    sh '''
                    echo 'Hello test stage'
                    #test -f build/index.html
                    npm test

                    '''
                }
                post{
                always{
                    junit 'jest-results/junit.xml'
                    
                }
                    }

                }
                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    
                    steps{
                    sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                    '''
                }
                 post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
                    }
     
                         }
                            }


    }
        
       


            stage('Deploy Staging'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                  
                    
                    steps{
                    sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging SITE_ID= $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                    '''
                }
                 post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
                    }
     
                         } 




        stage('Approve Deploy Production'){
            steps{
            timeout(1) {
            input cancel: 'Onaylamıyorum', message: 'Production ortamına deploy etmek üzeresiniz ', ok: 'Onaylıyorum'
            }
            }
        }



        




       
        stage('Deploy Production'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment{
                        CI_ENVIRONMENT_URL = 'https://polite-pony-6a9d41.netlify.app'
                    }
                    
                    steps{
                    sh '''
                    node --version
                    npm install netlify-cli@20.1.1 
                    node_modules/.bin/netlify --version
                    echo "Deploying to production SITE_ID= $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                    '''
                }
                 post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
                    }
     
                         } 
    
    }
    
}
