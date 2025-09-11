pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Dependencies') {
      steps { bat 'npm ci' }
    }

    stage('Test') {
      steps {
        bat 'if not exist reports mkdir reports'
        script {
          // Capture exit code so stage/pipe won't FAIL hard
          def code = bat(returnStatus: true, script: 'cmd /c npm test 1>reports\\test.log 2>&1')
          if (code != 0) {
            echo "Tests failed with exit code ${code}"
            currentBuild.result = 'UNSTABLE'
          }
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/test.log', allowEmptyArchive: true, fingerprint: true
          emailext(
            subject: "🧪 Test – ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
            to: "ayodyaekanayaka8@gmail.com",
            body: """Test stage finished: ${currentBuild.currentResult}
Build URL: ${env.BUILD_URL}""",
            attachmentsPattern: 'reports/test.log'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        bat 'if not exist reports mkdir reports'
        // Make sure npm audit never fails the stage
        bat 'cmd /c npm audit --json 1>reports\\npm-audit.json 2>&1 || exit /b 0'
        bat 'cmd /c npm audit 1>reports\\npm-audit.txt 2>&1 || exit /b 0'
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/npm-audit.*', allowEmptyArchive: true, fingerprint: true
          emailext(
            subject: "🔐 Security Scan – ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
            to: "ayodyaekanayaka8@gmail.com",
            body: """Security scan completed: ${currentBuild.currentResult}
Build URL: ${env.BUILD_URL}""",
            attachmentsPattern: 'reports/npm-audit.*'
          )
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true, fingerprint: true
      emailext(
        subject: "🏁 ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
        to: "ayodyaekanayaka8@gmail.com",
        body: """Build finished: ${currentBuild.currentResult}
Job: ${env.JOB_NAME}
Build URL: ${env.BUILD_URL}
Console: ${env.BUILD_URL}console""",
        attachmentsPattern: 'reports/**',
        attachLog: true
      )
    }
  }
}
