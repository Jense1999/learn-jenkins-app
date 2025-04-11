pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18'
                    args '-u root'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    apk add --no-cache python3 make g++ 
                    && npm ci
                    && npm run build
                    ls -la
                '''
            }
        }
        stage('Test') {
            steps{
                echo 'Test stage'
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }
    }
}