pipeline {
    agent any

    environment {
        NODE_IMAGE = 'node:18-alpine'
        PLAYWRIGHT_IMAGE = 'mcr.microsoft.com/playwright:v1.39.0-jammy'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Build') {
            agent {
                docker {
                    image "${NODE_IMAGE}"
                    args '-u node'
                }
            }
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image "${NODE_IMAGE}"
                            args '-u node'
                        }
                    }
                    steps {
                        sh '''
                        npm test -- --ci --reporters=default --reporters=jest-junit
                        '''
                        junit '**/junit.xml'
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image "${PLAYWRIGHT_IMAGE}"
                        }
                    }
                    steps {
                        sh '''
                        npm install -g serve
                        serve -s build & sleep 10
                        npx playwright test --reporter=html
                        '''
                        // Archive le rapport HTML
                        archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo 'Déploiement...'
                // Ici tu peux ajouter ton script de déploiement
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline terminé avec succès !'
        }
        failure {
            echo 'Pipeline échoué.'
        }
    }
}
