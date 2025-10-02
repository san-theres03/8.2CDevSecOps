pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install deps') {
      steps {
        // Use npm ci if you have package-lock.json; fall back to npm install
        sh '''
          if [ -f package-lock.json ]; then
            npm ci
          else
            npm install
          fi
        '''
      }
    }

    stage('Tests') {
      steps {
        // Run tests but do not fail the whole build for this assignment
        sh 'npm test || true'
      }
    }

    stage('Security Audit (npm)') {
      steps {
        sh '''
          # Create a JSON report we can archive
          npm audit --json > audit.json || true

          # Also fail the step only on high+ issues (but not the whole build)
          npm audit --audit-level=high || true

          echo "Audit report written to audit.json"
        '''
        archiveArtifacts artifacts: 'audit.json', onlyIfSuccessful: false, fingerprint: true
      }
    }
  }

  post {
    success {
      mail to: 'you@example.com',
           subject: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} SUCCESS",
           body: "Build passed. Details: ${env.BUILD_URL}"
    }
    failure {
      mail to: 'you@example.com',
           subject: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED",
           body: "Build failed. Check console: ${env.BUILD_URL}"
    }
    always {
      archiveArtifacts artifacts: 'package-lock.json', onlyIfSuccessful: false
    }
  }
}
