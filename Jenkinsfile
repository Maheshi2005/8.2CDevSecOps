    stage('SonarCloud Analysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SC_TOKEN')]) {
          powershell '''
            $ErrorActionPreference = "Stop"
            New-Item -ItemType Directory -Force -Path .sonar/tools | Out-Null
            Set-Location .sonar/tools

            $scVersion = "5.0.1.3006"
            $zipName = "sonar-scanner-cli-$scVersion-windows.zip"
            if (!(Test-Path "sonar-scanner-$scVersion-windows")) {
              Invoke-WebRequest -Uri "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/$zipName" -OutFile $zipName
              Expand-Archive -LiteralPath $zipName -DestinationPath .
              Remove-Item $zipName
            }

            $scannerDir = Get-ChildItem -Directory | Where-Object { $_.Name -like "sonar-scanner-*-windows" } | Select-Object -First 1
            $env:PATH = "$($scannerDir.FullName)\\bin;$env:PATH"

            Set-Location ../..
            sonar-scanner -D"sonar.login=$env:SC_TOKEN"
          '''
        }
      }
    }
