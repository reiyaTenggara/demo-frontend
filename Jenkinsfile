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
  tools {nodejs "node:18.16.0"}

  environment {
    SLACK_WORKSPACE = credentials('SLACK_WORKSPACE')
    SONARQUBE_LINK_GLOBAL = credentials('SONARQUBE_LINK_GLOBAL')
    REPOSITORY_NAME = sh(returnStdout: true, script: 'echo ${JOB_NAME} | cut -d "/" -f1').trim()
    HOME = '.'
  }

  options {
    disableConcurrentBuilds()
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(daysToKeepStr: "7",artifactDaysToKeepStr: "7"))
  }

  stages {

    stage('Notify Slack Channel') {
      steps {
        // get custom git log
        script {
            env.GITLOG = sh(returnStdout: true, script: 'git log --format="Commit ID: %h | Author: %an | Commit Message: %s" -1')
        }

        sendSlackNotification("Jenkins Pipeline initiated for Repository: ${REPOSITORY_NAME} | Branch: ${env.BRANCH_NAME} | Jenkins Build Number: ${env.BUILD_NUMBER}\n" +
        "${env.GITLOG}" +
        "<${env.BUILD_URL}console|View Console Output> || <${env.JOB_URL}|View Jobs Dashboard> || <${env.JOB_DISPLAY_URL}/${env.BRANCH_NAME}| View Blue Ocean Dashboard>")
      }
    }

    stage ('run coverage'){
      steps{
        sh 'npm install'
        sh 'npm run coverage'
      }
    }

    stage ('run parallel'){

      parallel {

        stage('Performing SonarQube Analysis') {
          environment {
            SCANNERHOME = tool 'SONARSCANNER'
          }
          steps {
            withSonarQubeEnv('SONARQUBE_SERVER') {
              sh """
              ${SCANNERHOME}/sonar-scanner-4.8.0.2856-linux/bin/sonar-scanner \
              -D sonar.projectKey=${REPOSITORY_NAME}:${env.BRANCH_NAME} \
              -D sonar.projectName=${REPOSITORY_NAME}:${env.BRANCH_NAME} \
              """
            }
            sendSlackNotification("The SonarQube scan has completed successfully. <${SONARQUBE_LINK_GLOBAL}${REPOSITORY_NAME}%3A${env.BRANCH_NAME}|Please check the code quality by clicking on this link>")
          }
        }

        stage('Build') {
          steps {
            withCredentials([string(credentialsId: "${env.BRANCH_NAME}_SERVER", variable: 'SERVER')]) {
              sshagent(credentials: ['PRIVATE_KEY']){
              sh"""
              ssh -o StrictHostKeyChecking=no ${SERVER} << EOF

              echo cd ~/micuci/micuci-fe
              cd ~/micuci/micuci-fe

              echo git pull
              git pull origin ${env.BRANCH_NAME}

              echo npm install
              export NVM_DIR=~/.nvm
              source ~/.nvm/nvm.sh
              npm install

              echo npm run build
              npm run build

              exit
              EOF"""
              }
            }
          }
        }
      }

    }
  }

  post {
    failure {
      sendSlackNotification('The build process has failed. Please review the build log and contact the administrator for assistance.', 'danger')
    }
    aborted {
      sendSlackNotification('The build process has been manually aborted.', 'warning')
    }
    success {
      sendSlackNotification('The build process has completed successfully.', 'good')
    }
  }

}