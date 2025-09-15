pipeline {
  agent any
  options { timestamps() }

  environment {
    // ID must match what you created in Jenkins > Credentials
    SNYK_TOKEN = credentials('SNYK_TOKEN')
  }

  stages {
    stage('Install Dependencies') {
      steps {
        sh 'npm ci || npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // Snyk CLI will pick up SNYK_TOKEN from env automatically.
        // If you ever need to force auth: sh 'npx snyk auth $SNYK_TOKEN || true'
        sh 'npm test || true'
      }
    }

    stage('Security Audit') {
      steps {
        sh 'npm audit || true'
      }
    }

    // Do archiving inside a stage (runs on the node so FilePath is available)
    stage('Archive Artifacts') {
      steps {
        script {
          if (fileExists('coverage'))       { archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true }
          if (fileExists('logs'))           { archiveArtifacts artifacts: 'logs/**',     allowEmptyArchive: true }
          if (fileExists('npm-debug.log'))  { archiveArtifacts artifacts: '**/npm-debug.log', allowEmptyArchive: true }
        }
      }
    }
  }
}

post {
  success {
    emailext to: 'sand.car2024@gmail.com',
             subject: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} success",
             body: "Build: ${env.BUILD_URL}\nCommit: ${env.GIT_COMMIT}"
  }
  failure {
    emailext to: 'sand.car2024@gmail.com',
             subject: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} failed",
             body: "Console: ${env.BUILD_URL}console"
  }
}

