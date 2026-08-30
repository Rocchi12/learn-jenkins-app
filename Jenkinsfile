pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'linux'
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }
        stage('Tests'){
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'linux'
            }
            }
            steps {
                sh '''
                    npm ci
                    npm test
                '''
            }
            post {
                always {
                    junit 'test-results/junit.xml'
                }
            }
        }
        stage('E2E'){
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:1.39.0-jammy'
                    label 'linux'
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    npm ci
                    npm i serve
                    node_modules/.bin/serve -s build & sleep 10
                    npx playwright test
                '''
            }
        }
    }
}
