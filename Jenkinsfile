pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'linux'
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
                sh 'chmod -R a+rwX build'
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
                            args '-u root:root'
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
                        }
                    }
                    steps {
                        deleteDir()
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
    }
}