pipeline {
  agent any
  options { timestamps() }

  environment {
    NOTIFY = 'ayodyaekanayaka8@gmail.com'   // <- change if needed
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm ci'
      }
    }

    stage('Test') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat '''
            if not exist reports mkdir reports
            npm test 1>reports\\test.log 2>&1
          '''
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/test.log', onlyIfSuccessful: false
        }
        success {
          emailext(
            to: env.NOTIFY,
            subject: '8.2CDevSecOps | Test stage: SUCCESS',
            body: '<p>Stage: <b>Test</b></p><p>Status: <b>SUCCESS</b></p><p>See attached test log and console for details.</p>',
            attachLog: true,
            attachmentsPattern: 'reports/test.log'
          )
        }
        failure {
          emailext(
            to: env.NOTIFY,
            subject: '8.2CDevSecOps | Test stage: FAILURE',
            body: '<p>Stage: <b>Test</b></p><p>Status: <b>FAILURE</b></p><p>See attached test log and console for details.</p>',
            attachLog: true,
            attachmentsPattern: 'reports/test.log'
          )
        }
      }
    }

    stage('Security Scan') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat '''
            if not exist reports mkdir reports
            cmd /c npm audit --json 1>reports\\npm-audit.json 2>&1
            cmd /c npm audit        1>reports\\npm-audit.txt  2>&1
            type reports\\npm-audit.txt
          '''
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'reports/npm-audit.*', onlyIfSuccessful: false
        }
        success {
          emailext(
            to: env.NOTIFY,
            subject: '8.2CDevSecOps | Security Scan stage: SUCCESS',
            body: '<p>Stage: <b>Security Scan</b></p><p>Status: <b>SUCCESS</b></p><p>Find results attached and in console.</p>',
            attachLog: true,
            attachmentsPattern: 'reports/npm-audit.*'
          )
        }
        failure {
          emailext(
            to: env.NOTIFY,
            subject: '8.2CDevSecOps | Security Scan stage: FAILURE',
            body: '<p>Stage: <b>Security Scan</b></p><p>Status: <b>FAILURE</b></p><p>Find results attached and in console.</p>',
            attachLog: true,
            attachmentsPattern: 'reports/npm-audit.*'
          )
        }
      }
    }
  }
}
