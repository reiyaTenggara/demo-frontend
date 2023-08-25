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
    SCANNERHOME = tool 'SONARSCANNER'
    REPOSITORY_NAME = sh(returnStdout: true, script: 'echo ${JOB_NAME} | cut -d "/" -f1')
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
        "🟡 Starting CI/CD for ${env.JOB_NAME}\n ${SCANNERHOME} " +
        "Build Number: ${env.BUILD_NUMBER} ${env.GITLOG}" +
        "<${env.BUILD_URL}console|Console Output> || <${env.JOB_URL}|Jobs Dashboard> || <${SONARQUBE_LINK_GLOBAL}${REPOSITORY_NAME}%3A${env.BRANCH_NAME}|Sonarqube>")
      }
    }

    stage('Performing SonarQube Analysis') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          withSonarQubeEnv('SONARQUBE_SERVER') {
            sh """
              ${SCANNERHOME}/sonar-scanner-4.8.0.2856-linux/bin/sonar-scanner \
              -D sonar.projectKey=${REPOSITORY_NAME}:${env.BRANCH_NAME} \
              -D sonar.projectName=${REPOSITORY_NAME}:${env.BRANCH_NAME} \
            """
          }
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