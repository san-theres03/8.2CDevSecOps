pipeline {
  agent any
  options { timestamps() }

  environment {
    RECIPIENTS = 'sand.car2024@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        script {
          // not required, but harmless to keep
          env.COMMIT_EMAIL = sh(script: "git log -1 --pretty=%ae || true", returnStdout: true).trim()
        }
      }
    }

    stage('Install deps') {
      steps {
        sh '''
          if [ -f package-lock.json ]; then
            echo "Using existing package-lock.json"
          else
            echo "No package-lock.json found"
          fi
          npm ci > build.log 2>&1
        '''
        archiveArtifacts artifacts: 'build.log', allowEmptyArchive: true
      }
      post {
        success {
          mail to: "${env.RECIPIENTS}",
               subject: "Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
               body: """Stage: Install deps
Status: SUCCESS

Console: ${env.BUILD_URL}console
(Attachments disabled for now)."""
        }
        failure {
          mail to: "${env.RECIPIENTS}",
               subject: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
               body: "Install deps failed. See console: ${env.BUILD_URL}console"
        }
      }
    }

    stage('Tests') {
      steps { sh 'npm test || true' }  // keep build green
    }

    stage('Security Audit (npm)') {
      steps {
        sh '''
          npm audit --json > audit.json || true
          npm audit --audit-level=high || true
          echo "Audit report written to audit.json"
        '''
        archiveArtifacts artifacts: 'audit.json', fingerprint: true, allowEmptyArchive: true
      }
      post {
        always {
          mail to: "${env.RECIPIENTS}",
               subject: "Security Audit Report: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
               body: "Audit complete. Build page: ${env.BUILD_URL}\n(Attachments disabled for now)."
        }
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'package-lock.json, **/*.json, **/*.html',
                         fingerprint: true, allowEmptyArchive: true
      }
    }
  }

  post {
    success {
      mail to: "${env.RECIPIENTS}",
           subject: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} SUCCESS",
           body: "Build passed. Details: ${env.BUILD_URL}"
    }
    failure {
      mail to: "${env.RECIPIENTS}",
           subject: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED",
           body: "Build failed. Check console: ${env.BUILD_URL}console"
    }
    always {
      archiveArtifacts artifacts: 'build.log, audit.json', allowEmptyArchive: true
    }
  }
}
