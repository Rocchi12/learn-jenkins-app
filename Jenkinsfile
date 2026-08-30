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
        }
    }
}
