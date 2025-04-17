pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '7cd8e834-5a8a-4603-a07d-36c23bfd3c5c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--network=host -u root --entrypoint=''"
                }
            }

            environment {
                AWS_S3_BUCKET = 'learn-jenkins-202504171045'
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        echo "Hello S3!" > index.html
                        aws s3 cp index.html s3://$AWS_S3_BUCKET/index.html
                    '''
                }
            }
        }


        stage('Build') {
            // This is a comment
            agent {
                docker {
                    image 'node:18-alpine'
                    args '--network=host -u root'
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

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '--network=host -u root'
                            reuseNode true
                        }
                    }

                    steps{
                        sh '''
                            test -f build/index.html
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
                            args '--network=host -u root:root'
                            reuseNode true
                        }
                    }

                    steps{
                        sh '''
                            serve -s build &
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

        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    args '--network=host -u root:root'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
            }

            steps{
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwright'
                    args '--network=host -u root:root'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://magenta-banoffee-135a34.netlify.app'
            }

            steps{
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}