pipeline {
  agent any
  options { timestamps() }

  environment {
    SNYK_TOKEN = credentials('snyk-token')   // uses the credential you add in Jenkins
  }

  stages {
    stage('Install Dependencies') {
      steps { sh 'npm ci || npm install' }
    }
    stage('Run Tests') {
      steps {
        // If you want explicit Snyk: sh 'npx snyk test || true'
        sh 'npm test || true'
      }
    }
    stage('Security Audit') {
      steps { sh 'npm audit || true' }
    }
  }

  post {
    always {
      script {
        // Only archive if something exists, avoids FilePath errors on early aborts
        if (fileExists('coverage') || fileExists('npm-debug.log') || fileExists('logs')) {
          archiveArtifacts artifacts: 'coverage/**, **/npm-debug.log, logs/**', allowEmptyArchive: true
        }
      }
    }
  }
}
