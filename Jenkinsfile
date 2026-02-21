pipeline {
    agent any

    environment {
        /*NETLIFY_SITE_ID = 'b67f0cec-a3f8-44b5-b7d3-d54af82b9a70' #netlify-token*/
        NETLIFY_SITE_ID = 'b85a4eff-dc89-4362-9cb8-89ad65d97136' /*#netlify-token2*/
        /*NETLIFY_AUTH_TOKEN = credentials('netlify-token') #netlify-token*/
        NETLIFY_AUTH_TOKEN = credentials('netlify-token2') /*netlify-token2*/
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la  # List files to verify we're in the correct directory
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -al
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 5  # Wait for the server to start
                            npx playwright test  --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true, includes: '**/*'])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            
            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy_output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy_output.json)
                    npx playwright test  --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging Local E2E', reportTitles: '', useWrapperFileDirectly: true, includes: '**/*'])
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            
            environment {
                /*CI_ENVIRONMENT_URL = 'https://splendid-marigold-1b2908.netlify.app' #netlify-token */
                CI_ENVIRONMENT_URL = 'https://inquisitive-phoenix-09ee8d.netlify.app' /*#netlify-token2*/
            }

            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    sleep 4
                    npx playwright test  --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true, includes: '**/*'])
                }
            }
        }
    }
}
