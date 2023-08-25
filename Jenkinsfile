// Send message to slack
def sendSlackNotification(String message, color='good') {
  slackSend(
      channel: '#cicd-notification',
      teamDomain: "${SLACK_WORKSPACE}",
      tokenCredentialId: 'SLACK_TOKEN',
      message: message,
      color: color
    )
}

pipeline {

  agent any
  tools {nodejs "node:16.16.0"}

  environment {
    SLACK_WORKSPACE = credentials('SLACK_WORKSPACE')
    SONARQUBE_LINK_GLOBAL = credentials('SONARQUBE_LINK_GLOBAL')
    HOME = '.'
    GITLOG = sh(returnStdout: true, script: 'git log --format="Author: %an | Commit ID: %h\n Commit Message: %s" -1')
    SCANNERHOME = tool 'SONAR_NEW'
  }

  options {
    disableConcurrentBuilds()
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(daysToKeepStr: "7",artifactDaysToKeepStr: "7"))
  }

  stages {

    stage('Notify Slack Channel') {
      steps {
        sendSlackNotification(
        "🟡 Starting CI/CD for ${env.JOB_NAME}\n " +
        "Build Number: ${env.BUILD_NUMBER} ${env.GITLOG}" +
        "<${env.BUILD_URL}console|Console Output> || <${env.JOB_URL}|Jobs Dashboard> || <${env.JOB_DISPLAY_URL}/${env.BRANCH_NAME}|Blue Ocean Dashboard> || <${SONARQUBE_LINK_GLOBAL}${env.JOB_NAME}|Sonarqube>")
      }
    }

    stage('Performing SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SONARQUBE_SERVER') {
          sh """
          ${SCANNERHOME}/bin/sonar-scanner \
          -D sonar.projectKey=${env.JOB_NAME} | tr '/' ':'
          """
        }        
      }
    }


    stage('Install Node.js Packages (npm install)') {
      steps {
        sh 'rm -rf package-lock.json'
        sh 'npm install'
      }
    }

    stage('Build Node.js Project (npm run build)') {
      steps {
        sh 'npm run build'
      }
    }


  }

  post {
    failure {
      sendSlackNotification('🟡 The build process has failed.', 'danger')
    }
    aborted {
      sendSlackNotification('🟡 The build process has been manually aborted.', 'warning')
    }
    success {
      archiveArtifacts artifacts: 'dist/**', onlyIfSuccessful: true
      sendSlackNotification('🟡 The build process has completed successfully.', 'good')
    }
  }
}