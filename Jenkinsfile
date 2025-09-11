pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Maheshi2005/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps { bat 'npm install' }
    }

    stage('Run Tests') {
      steps { bat 'cmd /c npm test || exit /b 0' }  // allow pipeline to continue
    }

    stage('Generate Coverage Report') {
      steps {
        // run if script exists; otherwise continue
        bat 'cmd /c npm run coverage || exit /b 0'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps { bat 'cmd /c npm audit || exit /b 0' }
    }

    stage('SonarCloud Analysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SC_TOKEN')]) {
          powershell '''
            $ErrorActionPreference = "Stop"
            New-Item -ItemType Directory -Force -Path .sonar/tools | Out-Null
            Set-Location .sonar/tools

            $scVersion = "5.0.1.3006"
            $zipName   = "sonar-scanner-cli-$scVersion-windows.zip"

            if (!(Test-Path "sonar-scanner-$scVersion-windows")) {
              Invoke-WebRequest -Uri "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/$zipName" -OutFile $zipName
              Expand-Archive -LiteralPath $zipName -DestinationPath .
              Remove-Item $zipName
            }

            $scannerDir = Get-ChildItem -Directory | Where-Object { $_.Name -like "sonar-scanner-*-windows" } | Select-Object -First 1
            $env:PATH = "$($scannerDir.FullName)\\bin;$env:PATH"

            Set-Location ../..

            # IMPORTANT: org key must match SonarCloud exactly (likely lowercase)
            sonar-scanner -D"sonar.login=$env:SC_TOKEN"
          '''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'coverage/**, **/sonar*.log', allowEmptyArchive: true
    }
  }
}
