def sendSlackNotification(message, color='good') {
    slackSend (
            channel: '#jenkins',
            teamDomain: "$slack_workspace_local",
            tokenCredentialId: "slack_token_local",
            message: message,
            color: color
    )
}

pipeline {

    environment {
        slack_workspace_local = credentials('slack_workspace_local')
    }

    agent any
    tools {
        nodejs 'node18'
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 20, unit: 'MINUTES')
    }
    stages {

        stage('Send Slack Notificationaa') {
            steps {
                sendSlackNotification("Starting CI/CD on job: ${env.JOB_NAME}\n"+
                    "Build Number: ${env.BUILD_NUMBER}\n" +
                    "<${env.BUILD_URL}console|View Output> || <${env.JOB_URL}|View Job> || <${env.JOB_DISPLAY_URL}/${env.BRANCH_NAME}| Open Blue Ocean>")
            }
        }

        stage('Npm install & build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Upload dist folder to S3 bucket') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                s3Upload (
                        consoleLogLevel: 'WARNING',
                        dontSetBuildResultOnFailure: true,
                        dontWaitForConcurrentBuildCompletion: false,
                        entries: [
                                [
                                bucket: 'demo-frontend123',
                                excludedFile: '',
                                flatten: false,
                                gzipFiles: false,
                                keepForever: false,
                                managedArtifacts: false,
                                noUploadOnFailure: true,
                                selectedRegion: 'ap-southeast-1',
                                showDirectlyInBrowser: false,
                                sourceFile: 'dist/**',
                                storageClass: 'STANDARD',
                                uploadFromSlave: false,
                                        useServerSideEncryption: false
                                ]
                        ],
                        pluginFailureResultConstraint: 'FAILURE',
                        profileName: 'demo-frontend123',
                        userMetadata: []
                )
            }
        }
    }

    post {
        aborted {
            sendSlackNotification("Buasdfild Aborted", "warning")
        }
        failure {
            sendSlackNotification("Build Failed", "danger")

        }
        success {
            sendSlackNotification("Build Success", "good")
        }
    }
}
