pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '1cae29e8-09a1-4b1b-8fec-2a4d40020948'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        // This is a comment
        /*
        line 1
        line 2
        */
        stage('Build') {
            agent {
                    docker {
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
        
        stage ('Run Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'echo "Test stage"'
                        sh 'test -f build/index.html'
                        sh 'npm ci'
                        sh 'npm test'
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        
                        }
                    }
                    steps {
                        sh '''
                            npm ci
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
            
                }
            }
        }

        stage('Deploy') {
            agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
            }
            steps {
                sh '''
                   npm install netlify-cli@20.1.1
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                
                }
            }

        environment {
            CI_ENVIRONMENT_URL = 'https://frabjous-alpaca-f52f40.netlify.app'
        }

            steps {
                sh '''
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
