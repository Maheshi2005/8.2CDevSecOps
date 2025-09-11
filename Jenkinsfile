pipeline {
  agent any
  options { timestamps() }

  environment {
    MAIL_TO = 'ayodyaekanayaka8@gmail.com'   // <-- change if needed
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
          // Run tests; if they fail, mark stage FAILURE but keep pipeline going (build becomes UNSTABLE)
          catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
            bat '''
              if not exist reports mkdir reports
              npm test > reports\\test.log 2>&1
            '''
          }
        }
      }
      post {
        always {
          emailext(
            to: env.MAIL_TO,
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} – TEST stage: ${currentBuild.currentResult}",
            body: """<p><b>Test</b> stage finished with: <b>${currentBuild.currentResult}</b>.</p>
                     <p>Job: ${env.JOB_NAME} | Build: #${env.BUILD_NUMBER}</p>""",
            mimeType: 'text/html',
            attachLog: true,
            attachmentsPattern: 'reports/test.log'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        script {
          // Run npm audit; never abort pipeline even if it finds vulnerabilities
          catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
            bat '''
              if not exist reports mkdir reports
              cmd /c npm audit --json > reports\\npm-audit.json 2>&1
              cmd /c npm audit > reports\\npm-audit.txt 2>&1
            '''
          }
        }
      }
      post {
        always {
          emailext(
            to: env.MAIL_TO,
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} – SECURITY SCAN: ${currentBuild.currentResult}",
            body: """<p><b>Security Scan</b> finished with: <b>${currentBuild.currentResult}</b>.</p>
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
