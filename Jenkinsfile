pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Maheshi2005/8.2CDevSecOps.git'
      }
    }
    stage('Install Dependencies') { steps { bat 'npm install' } }
    stage('Run Tests')            { steps { bat 'npm test || exit /b 0' } }
    stage('Generate Coverage')    { steps { bat 'npm run coverage || exit /b 0' } }
    stage('NPM Audit (Security)') { steps { bat 'npm audit || exit /b 0' } }
  }
}
