pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '5c1ad21b-6377-4545-a29b-02fc88c589ff'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit test') {
                    steps {
                        sh 'npm test'
                        junit 'jest-results/junit.xml'
                    }
                }
                stage('E2E') {
                    steps {
                        sh '''
                            npm install serve
                            ./node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                }
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report', // dossier généré par Playwright
                        reportFiles: 'index.html',
                        reportName: 'Rapport des Tests'
                    ])
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    npm install netlify-cli -g
                    netlify --version
                    echo "Deploying to Netlify site: $NETLIFY_SITE_ID"
                    netlify status
                '''
            }
        }
    }
}
