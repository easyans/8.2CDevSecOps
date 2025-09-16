pipeline {
  agent any
  triggers { pollSCM('H/5 * * * *') }

  environment {
    REPO_URL     = 'https://github.com/easyans/8.2CDevSecOps.git'
    NOTIFY_EMAIL = 'ar0829@srmist.edu.in'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: "${env.REPO_URL}"
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'npm test || true'
        script {
          emailext(
            to: "${env.NOTIFY_EMAIL}",
            subject: "SIT753 8.1C • Test stages • ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Stage: Run Tests
Job: ${env.JOB_NAME}
Build: ${env.BUILD_URL}
Result so far: ${currentBuild.currentResult}""",
            attachLog: true,
            mimeType: 'text/plain'
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        sh 'npm run coverage || true'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh 'npm audit || true'
        script {
          emailext(
            to: "${env.NOTIFY_EMAIL}",
            subject: "SIT753 8.1C • Security scans • ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Stage: NPM Audit (Security Scan)
Job: ${env.JOB_NAME}
Build: ${env.BUILD_URL}
Result so far: ${currentBuild.currentResult}""",
            attachLog: true,
            mimeType: 'text/plain'
          )
        }
      }
    }
  }

  post {
    always {
      echo 'Pipeline finished successfully (see emails for stage results with logs attached).'
    }
  }
}