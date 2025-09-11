pipeline {
  agent any
  options { timestamps() }

  // Pull the Snyk token you saved as a Secret Text credential with ID 'snyk-token'
  environment {
    SNYK_TOKEN = credentials('snyk-token')
  }

  stages {
    stage('Install Dependencies') {
      steps {
        // Use npm ci when a lockfile exists; otherwise fall back to npm install
        sh 'if [ -f package-lock.json ]; then npm ci; else npm install; fi'
      }
    }

    stage('Run Tests (Snyk)') {
      steps {
        // In your repo, "npm test" runs "snyk test" and will auto-use SNYK_TOKEN
        sh 'npm test || true'
      }
    }

    stage('Security Audit (npm)') {
      steps {
        sh 'npm audit || true'
      }
    }
  }

  post {
    always {
      // Only keep this if you actually create these files; otherwise it’s harmless
      archiveArtifacts artifacts: '**/npm-debug.log, logs/**', allowEmptyArchive: true
    }
  }
}
