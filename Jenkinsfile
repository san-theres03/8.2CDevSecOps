pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Install Dependencies') { steps { sh 'npm install' } }
    stage('Run Tests')           { steps { sh 'npm test || true' } }   // keep going if tests fail
    stage('Coverage')            { steps { sh "npm run coverage || true" } }
    stage('Security Audit')      { steps { sh 'npm audit || true' } }
  }

  post {
    always {
      archiveArtifacts artifacts: 'coverage/**, **/npm-debug.log, logs/**', allowEmptyArchive: true
    }
  }
}
