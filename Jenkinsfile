pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '92612489-2797-41ff-a54b-1d06a7717ab3'
        NETLIFY_AUTH_TOKEN = credentials('net-tkn')
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'linux'
                    args '-u 1000:1000'
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
                stash name: 'build-output', includes: 'build/**'
            }
        }
        stage('Run Tests'){
            parallel {
                stage('Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            label 'linux'
                            args '-u 1000:1000'
                        }
                    }
                    steps {
                        sh '''
                            npm ci
                            npm test -- --ci --reporters=default --reporters=jest-junit
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            label 'linux'
                            args '-u 1000:1000'
                        }
                    }
                    steps {
                        unstash 'build-output'
                        sh '''
                            npm ci
                            npm i serve wait-on
                            node_modules/.bin/serve -s build &
                            npx wait-on http://localhost:3000
                            npx playwright test
                        '''
                    }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'linux'
                    args '-u 1000:1000'
                }
            }
            steps {
                unstash 'build-output'
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod --json > deploy-output.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                '''
            }
        }
    }
}