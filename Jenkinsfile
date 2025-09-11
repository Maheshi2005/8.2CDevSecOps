pipeline {
    agent any

    options { timestamps() }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Test') {
            steps {
                bat 'if not exist reports mkdir reports'
                // run tests and write output to log file
                bat 'npm test 1>reports\\test.log 2>&1'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/test.log', allowEmptyArchive: true
                }
                success {
                    emailext(
                        to: 'ayodyaekanayaka8@gmail.com',
                        subject: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} – Test stage SUCCESS",
                        body: """Test stage succeeded.

Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
URL:  ${env.BUILD_URL}""",
                        attachmentsPattern: 'reports/test.log'
                    )
                }
                failure {
                    emailext(
                        to: 'ayodyaekanayaka8@gmail.com',
                        subject: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} – Test stage FAILED",
                        body: """Test stage failed.

Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
URL:  ${env.BUILD_URL}

See attached test log and console log.""",
                        attachmentsPattern: 'reports/test.log',
                        attachLog: true
                    )
                }
            }
        }

        stage('Security Scan') {
            steps {
                bat 'if not exist reports mkdir reports'
                // create both JSON + text logs
                bat 'cmd /c npm audit --json 1>reports\\npm-audit.json 2>&1'
                bat 'cmd /c npm audit        1>reports\\npm-audit.txt  2>&1'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/npm-audit.*', allowEmptyArchive: true
                }
                success {
                    emailext(
                        to: 'ayodyaekanayaka8@gmail.com',
                        subject: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} – Security Scan SUCCESS",
                        body: """Security Scan succeeded.

Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
URL:  ${env.BUILD_URL}""",
                        attachmentsPattern: 'reports/npm-audit.txt,reports/npm-audit.json'
                    )
                }
                failure {
                    emailext(
                        to: 'ayodyaekanayaka8@gmail.com',
                        subject: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} – Security Scan FAILED",
                        body: """Security Scan failed.

Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
URL:  ${env.BUILD_URL}

See attached audit logs and console log.""",
                        attachmentsPattern: 'reports/npm-audit.txt,reports/npm-audit.json',
                        attachLog: true
                    )
                }
            }
        }
    }
}
