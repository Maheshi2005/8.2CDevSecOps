pipeline {
  agent any
  options { timestamps() }

  environment {
    MAIL_TO = 'ayodyaekanayaka8@gmail.com' 
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
        bat '''
          if not exist reports mkdir reports
          npm test > reports\\test.log 2>&1
        '''
      }
      post {
        always {
          emailext(
            to: env.MAIL_TO,
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} – TEST stage: ${currentBuild.currentResult}",
            body: """<p>The <b>Test</b> stage has finished with status: <b>${currentBuild.currentResult}</b>.</p>
                     <p>Attached: console log and test log.</p>""",
            mimeType: 'text/html',
            attachLog: true,
            attachmentsPattern: 'reports/test.log'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        // Do not fail the whole build just because audit finds vulns; still email results
        bat '''
          if not exist reports mkdir reports
          cmd /c npm audit --json > reports\\npm-audit.json 2>&1
          cmd /c npm audit > reports\\npm-audit.txt 2>&1
        '''
      }
      post {
        always {
          emailext(
            to: env.MAIL_TO,
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} – SECURITY SCAN: ${currentBuild.currentResult}",
            body: """<p>The <b>Security Scan</b> stage has finished with status: <b>${currentBuild.currentResult}</b>.</p>
                     <p>Attached: npm-audit reports and console log.</p>""",
            mimeType: 'text/html',
            attachLog: true,
            attachmentsPattern: 'reports/npm-audit.*'
          )
        }
      }
    }
  }
}
