pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }

  environment {
    EMAIL_TO = 'ayodyaekanayaka8@gmail.com' // <-- change if needed
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
        bat 'if not exist reports mkdir reports'
        // Run tests; if they fail, mark the stage but keep the pipeline going
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
          bat 'cmd /c npm test  1>reports\\test.log 2>&1'
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/test.log', onlyIfSuccessful: false
        }
        success {
          emailext(
            to: "${env.EMAIL_TO}",
            subject: "TEST ✅ ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """<p>Stage <b>Test</b> finished: <b>SUCCESS</b></p>
                     <p>Console: ${env.BUILD_URL}console</p>""",
            attachmentsPattern: 'reports/test.log'
          )
        }
        unsuccessful {
          emailext(
            to: "${env.EMAIL_TO}",
            subject: "TEST ❌ ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """<p>Stage <b>Test</b> finished: <b>FAILED/UNSTABLE</b></p>
                     <p>Console: ${env.BUILD_URL}console</p>""",
            attachmentsPattern: 'reports/test.log'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        bat 'if not exist reports mkdir reports'
        // npm audit returns non-zero exit codes; catch them but continue
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
          bat 'cmd /c npm audit --json  1>reports\\npm-audit.json 2>&1'
          bat 'cmd /c npm audit        1>reports\\npm-audit.txt  2>&1'
          // propagate audit exit code into catchError
          bat 'cmd /c exit /b %ERRORLEVEL%'
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/npm-audit.*', onlyIfSuccessful: false
        }
        success {
          emailext(
            to: "${env.EMAIL_TO}",
            subject: "SECURITY ✅ ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """<p>Stage <b>Security Scan</b>: <b>SUCCESS</b></p>
                     <p>Console: ${env.BUILD_URL}console</p>""",
            attachmentsPattern: 'reports/npm-audit.*'
          )
        }
        unsuccessful {
          emailext(
            to: "${env.EMAIL_TO}",
            subject: "SECURITY ⚠️ ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """<p>Stage <b>Security Scan</b>: <b>FAILED/UNSTABLE</b></p>
                     <p>Console: ${env.BUILD_URL}console</p>""",
            attachmentsPattern: 'reports/npm-audit.*'
          )
        }
      }
    }
  }
}
