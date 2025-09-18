pipeline {
  agent any
  tools {
    nodejs 'Node18'  // Changed to match your Jenkins configuration
  }
  triggers { pollSCM('H/5 * * * *') }

  environment {
    REPO_URL     = 'https://github.com/easyans/8.2CDevSecOps.git'
    NOTIFY_EMAIL = 'aakashraj232002@gmail.com'
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
      }
      post {
        always {
          emailext(
            to: "${env.NOTIFY_EMAIL}",
            subject: "SIT753 8.1C • Test Results • ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Pipeline Stage: Run Tests
Status: ${currentBuild.currentResult}
Job: ${env.JOB_NAME}
Build: ${env.BUILD_URL}
Build Duration: ${currentBuild.durationString}""",
            attachLog: true,
            attachmentsPattern: '**/*.log',
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
        sh 'npm audit --json > npm-audit-report.json || true'
      }
      post {
        always {
          emailext(
            to: "${env.NOTIFY_EMAIL}",
            subject: "SIT753 8.1C • Security Scan Results • ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Pipeline Stage: NPM Audit (Security Scan)
Status: ${currentBuild.currentResult}
Job: ${env.JOB_NAME}
Build: ${env.BUILD_URL}
Vulnerabilities: Check attached reports for details""",
            attachLog: true,
            attachmentsPattern: '**/*-report.json, **/*.log',
            mimeType: 'text/plain'
          )
        }
      }
    }
  }

  post {
    always {
      echo 'Pipeline completed. Email notifications sent for test and security stages.'
    }
    failure {
      emailext(
        to: "${env.NOTIFY_EMAIL}",
        subject: "🚨 PIPELINE FAILED • ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Pipeline Status: FAILED
Job: ${env.JOB_NAME}
Build: ${env.BUILD_URL}
Failed Stage: ${currentBuild.currentResult}
Check Jenkins for details and logs""",
        attachLog: true,
        attachmentsPattern: '**/*.log'
      )
    }
    success {
      emailext(
        to: "${env.NOTIFY_EMAIL}",
        subject: "✅ PIPELINE IS SUCCESS • ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Pipeline Status: SUCCESS
Job: ${env.JOB_NAME}
Build: ${env.BUILD_URL}
Build Duration: ${currentBuild.durationString}
All stages completed successfully""",
        attachLog: false
      )
    }
  }
}
