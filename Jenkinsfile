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
    REPOSITORY_NAME = sh(returnStdout: true, script: 'echo ${JOB_NAME} | cut -d "/" -f1').trim()
    HOME = '.'
    GITLOG = sh(returnStdout: true, script: 'git log --format="Author: %an | Commit ID: %h\n Commit Message: %s" -1')
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
          env.GITLOG = sh(returnStdout: true, script: 'git log --format="Author: %an | Commit ID: %h\n Commit Message: %s" -1')
        }
        sendSlackNotification(
        "🟡 Starting CI/CD for ${env.JOB_NAME}\n" +
        "${env.GITLOG}" +
        "<${env.BUILD_URL}console|Console Output> || <${env.JOB_URL}|Jobs Dashboard> || <${env.JOB_DISPLAY_URL}/${env.BRANCH_NAME}|Blue Ocean Dashboard> || <${SONARQUBE_LINK_GLOBAL}${REPOSITORY_NAME}%3A${env.BRANCH_NAME}|Sonarqube>")
      }
    }

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

    stage('Inject Environment Variables') {
      steps {
        withCredentials([file(credentialsId: "${env.BRANCH_NAME}_ENV", variable: 'ENV'),]) {
          sh 'mv ${ENV} .env'
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

    stage('Upload Static Files to AWS S3 Bucket') {
      steps {
        withCredentials([string(credentialsId: "${env.BRANCH_NAME}_BUCKET_NAME", variable: 'BUCKET_NAME'),]) {
          withAWS(region:'ap-southeast-1', credentials:'AWS_CREDENTIAL') {
            s3Upload(file:'./dist', bucket: "${BUCKET_NAME}", path:'')
          } 
        }
      }
    }

  }

  // post {
  //   failure {
  //     sendSlackNotification('The build process has failed. Please review the build log and contact the administrator for assistance.', 'danger')
  //   }
  //   aborted {
  //     sendSlackNotification('The build process has been manually aborted.', 'warning')
  //   }
  //   success {
  //     archiveArtifacts artifacts: 'dist/**', onlyIfSuccessful: true
  //     sendSlackNotification('The build process has completed successfully.', 'good')
  //   }
  // }
}