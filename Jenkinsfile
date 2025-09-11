pipeline {
  agent any
  options { timestamps() }

  environment {
    // ======= change if you want to mail a different address =======
    EMAIL_TO = 'ayodyaekanayaka8@gmail.com'
  }

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
        script {
          // ensure reports dir exists
          bat 'if not exist reports mkdir reports'

          // run tests without failing the stage; capture exit code
          int rc = bat(returnStatus: true, script: 'npm test 1>reports\\test.log 2>&1')
          String stageStatus = (rc == 0) ? 'SUCCESS' : 'FAILURE'

          // mark build UNSTABLE on failure so pipeline continues
          if (rc != 0) { currentBuild.result = 'UNSTABLE' }

          // archive and email
          archiveArtifacts artifacts: 'reports/test.log', allowEmptyArchive: true
          emailext(
            to: EMAIL_TO,
            subject: "[${env.JOB_NAME}] ${env.STAGE_NAME}: ${stageStatus} (#${env.BUILD_NUMBER})",
            attachmentsPattern: 'reports/test.log',
            body: """\
Stage : ${env.STAGE_NAME}
Status: ${stageStatus}

Job   : ${env.JOB_NAME} #${env.BUILD_NUMBER}
URL   : ${env.BUILD_URL}

Attached: test.log
"""
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        script {
          bat 'if not exist reports mkdir reports'

          // write both JSON and text reports; capture return code from the text run
          bat 'cmd /c npm audit --json  1>reports\\npm-audit.json 2>&1'
          int rc = bat(returnStatus: true, script: 'cmd /c npm audit 1>reports\\npm-audit.txt 2>&1')
          bat 'type reports\\npm-audit.txt'  // print into console for visibility

          String stageStatus = (rc == 0) ? 'SUCCESS' : 'FAILURE'
          if (rc != 0) { currentBuild.result = 'UNSTABLE' }

          archiveArtifacts artifacts: 'reports/npm-audit.*', allowEmptyArchive: true
          emailext(
            to: EMAIL_TO,
            subject: "[${env.JOB_NAME}] ${env.STAGE_NAME}: ${stageStatus} (#${env.BUILD_NUMBER})",
            attachmentsPattern: 'reports/npm-audit.*',
            body: """\
Stage : ${env.STAGE_NAME}
Status: ${stageStatus}

Job   : ${env.JOB_NAME} #${env.BUILD_NUMBER}
URL   : ${env.BUILD_URL}

Attached: npm-audit.txt, npm-audit.json
"""
          )
        }
      }
    }
  }
}
