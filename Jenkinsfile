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
        // Save test console output to a file (Windows path escaping)
        bat 'npm test 1>reports\\test.log 2>&1'
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/test.log', allowEmptyArchive: true, fingerprint: true
          emailext(
            subject: "🧪 Test stage – ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
            to: "ayodyaekanayaka8@gmail.com",
            body: """Test stage finished with status: ${currentBuild.currentResult}
Project : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Console : ${env.BUILD_URL}""",
            attachmentsPattern: 'reports/test.log',
            attachLog: false
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        bat 'if not exist reports mkdir reports'
        // JSON + human-readable reports
        bat 'cmd /c npm audit --json 1>reports\\npm-audit.json 2>&1'
        bat 'cmd /c npm audit 1>reports\\npm-audit.txt 2>&1'
        bat 'type reports\\npm-audit.txt'
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/npm-audit.*', allowEmptyArchive: true, fingerprint: true
          emailext(
            subject: "🔒 Security Scan – ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
            to: "ayodyaekanayaka8@gmail.com",
            body: """Security scan finished with status: ${currentBuild.currentResult}
See attached npm audit report(s).
Console : ${env.BUILD_URL}""",
            attachmentsPattern: 'reports/npm-audit.*',
            attachLog: false
          )
        }
      }
    }
  }

  // Final summary email for the whole pipeline
  post {
    always {
      archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true, fingerprint: true
      emailext(
        subject: "✅ Build ${env.JOB_NAME} #${env.BUILD_NUMBER} – ${currentBuild.currentResult}",
        to: "ayodyaekanayaka8@gmail.com",
        body: """Pipeline finished with status: ${currentBuild.currentResult}

Job     : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Duration: ${currentBuild.durationString}
Console : ${env.BUILD_URL}

Artifacts from 'reports/' are attached/archived.
""",
        attachmentsPattern: 'reports/**',
        attachLog: true
      )
    }
  }
}
