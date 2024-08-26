pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = '0f85d9db-56b1-4026-8f3b-bffc519861c4'
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
                sh'''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                
                '''
            }
        }
        
        stage('Run Tests'){
            parallel{
                stage('Uni tests'){
                            agent{
                                docker{
                                    image 'node:18-alpine'
                                    reuseNode true
                                }
                            }
                            steps{
                                sh'''
                                    #test -f build/index.html
                                    npm test
                                '''
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
                        sh'''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10 
                        npx playwright test  --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }

                }
            }
           
        } 

        stage('Deploy staging') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh'''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json 
                 '''
            }
        } 
        stage('Approval') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    input message: 'Ready to deploy?', ok: 'Yes, I am sure to deploy!'
                }
            }
        }
        stage('Deploy prod') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh'''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --prod
                 '''
            }
        } 
        stage('Prod E2E'){
          
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://voluble-melba-f1564a.netlify.app'
            }
            steps{
                sh'''
                    npx playwright test  --reporter=html
                '''
            }
    
            post{
                always{
                junit 'results/junit.xml'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        } 
        

    }
    
}
