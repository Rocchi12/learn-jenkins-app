pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'linux'
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                cleanWs()
                checkout scm
                sh '''
                    npm ci
                    npm run build
                '''
                // Ensure stashed files are readable/writable by any UID that unstashes them
                sh 'chmod -R a+rwX build'
                stash name: 'build-output', includes: 'build/**'
            }
        }

        stage('Run Tests') {
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
                            junit testResults: 'test-results/junit.xml', allowEmptyResults: true
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            label 'linux'
                            args '-u root:root'
                        }
                    }
                    steps {
                        cleanWs()
                        checkout scm
                        unstash 'build-output'
                        sh '''
                            npm ci
                            npm i serve wait-on
                            node_modules/.bin/serve -s build &
                            npx wait-on http://localhost:3000
                            npx playwright test
                        '''
                    }
                    post {
                        always {
                            sh 'pkill -f "serve -s build" || true'
                        }
                    }
                }
            }
        }
    }
}