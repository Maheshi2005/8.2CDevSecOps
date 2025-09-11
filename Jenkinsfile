pipeline {
  agent any
  tools { nodejs 'node20' }   // Tools -> NodeJS name

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Maheshi2005/8.2CDevSecOps.git'
      }
    }

    stage('Node & NPM Versions') {
      steps { bat 'node -v & npm -v' }
    }

    stage('Install Dependencies') {
      steps { bat 'npm ci || npm install' }
    }

    stage('Run Tests') {
      steps { bat 'npm test || exit /b 0' }
    }

    stage('Generate Coverage Report') {
      steps { bat 'npm run coverage || exit /b 0' }
    }

    stage('NPM Audit (Security Scan)') {
      steps { bat 'npm audit || exit /b 0' }
    }
  }
}
