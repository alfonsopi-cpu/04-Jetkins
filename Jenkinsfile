pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
        timeout(time: 5, unit: 'MINUTES')
    }

    environment {
        FORCE_COLOR = '0'
        NO_COLOR = 'true'
    }

    stages {
        stage('Audit tools') {
            steps {
                dir('backend') {
                    sh 'node --version'
                }
            }
        }

        stage('Install dependencies') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }

        stage('Format check') {
            steps {
                dir('backend') {
                    sh 'npm run format:check'
                }
            }
        }

        stage('Code quality') {
            steps {
                dir('backend') {
                    sh 'npm run lint'
                }
            }
        }

        stage('Type check') {
            steps {
                dir('backend') {
                    sh 'npm run type-check'
                }
            }
        }

        stage('Tests') {
            steps {
                dir('backend') {
                    sh 'npm run test'
                }
            }
        }

        stage('Build') {
            steps {
                dir('backend') {
                    sh 'npm run build'
                    archiveArtifacts artifacts: 'dist/**', fingerprint: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Review logs.'
        }

        always {
            cleanWs()
        }
    }
}