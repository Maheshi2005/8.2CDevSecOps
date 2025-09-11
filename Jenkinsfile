pipeline {
  agent any
  options { timestamps() }

  environment {
    EMAIL_TO = 'ayodyaekanayaka8@gmail.com'   // <-- change if needed
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Dependencies') {
      steps { bat 'npm ci' }
    }

    stage('Test') {
      steps {
        script {
          bat 'if not exist reports mkdir reports'

          int rc = bat(returnStatus: true,
                       script: 'npm test 1>reports\\test.log 2>&1')
          String stageStatus = (rc == 0) ? 'SUCCESS' : 'FAILURE'
          if (rc != 0) currentBuild.result = 'UNSTABLE'

          archiveArtifacts artifacts: 'reports/test.log', allowEmptyArchive: true
          emailext(
            to: EMAIL_TO,
            subject: "[${env.JOB_NAME}] ${env.STAGE_NAME}: ${stageStatus} (#${env.BUILD_NUMBER})",
            attachmentsPattern: 'reports/test.log',
            body: """Stage: ${env.STAGE_NAME}
Status: ${stageStatus}
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}

Attached: test.log"""
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        script {
          bat 'if not exist reports mkdir reports'

          // write JSON (ignored exit code) and text (capture exit code)
          bat(returnStatus: true,
              script: 'cmd /c npm audit --json 1>reports\\npm-audit.json 2>&1')
          int rc = bat(returnStatus: true,
                       script: 'cmd /c npm audit 1>reports\\npm-audit.txt 2>&1')
          bat 'type reports\\npm-audit.txt'  // show in console

          String stageStatus = (rc == 0) ? 'SUCCESS' : 'FAILURE'
          if (rc != 0) currentBuild.result = 'UNSTABLE'

          archiveArtifacts artifacts: 'reports/npm-audit.*', allowEmptyArchive: true
          emailext(
            to: EMAIL_TO,
            subject: "[${env.JOB_NAME}] ${env.STAGE_NAME}: ${stageStatus} (#${env.BUILD_NUMBER})",
            attachmentsPattern: 'reports/npm-audit.*',
            body: """Stage: ${env.STAGE_NAME}
Status: ${stageStatus}
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}

Attached: npm-audit.txt, npm-audit.json"""
          )
        }
      }
    }
  }
}
