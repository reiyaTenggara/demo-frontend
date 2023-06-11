pipeline {
    agent any
    tools {
        nodejs 'node18'
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 40, unit: 'MINUTES')
    }
    stages {

        stage('Send Slack Notificationaa') {
            steps {
                withCredentials([string(credentialsId: 'slack_workspace_global', variable: 'slack_workspace_global')]) {
                    slackSend (
                        channel: '#jenkins',
                        teamDomain: 'demo-olh3682',
                        tokenCredentialId: 'slack_token_local',
                        message: """
                    Starting CI/CD on job: ${env.JOB_NAME}
                    Build Number: ${env.BUILD_NUMBER}
                    <${env.BUILD_URL}console|View Output> || <${env.JOB_URL}|View Job> || <${env.JOB_DISPLAY_URL}/${env.BRANCH_NAME}| Open Blue Ocean>
                    """
                    )
                }

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
            slackSend (
                    channel: '#jenkins',
                    teamDomain: 'demo-olh3682',
                    tokenCredentialId: 'slack_token',
                    message: '''
                    Starting CI/CD on job: ${env.JOB_NAME}
                    Build Failure
                '''
            )
        }
        failure {
            slackSend (
                    channel: '#jenkins',
                    teamDomain: 'demo-olh3682',
                    tokenCredentialId: 'slack_token_local',
                    message: '''
                    Starting CI/CD on job: ${env.JOB_NAME}
                    Build Failure
                '''
            )
        }
        success {
            slackSend (
                    channel: '#jenkins',
                teamDomain: 'slack_workspace_local',
                tokenCredentialId: 'slack_token_local',
                message: """
                Starting CI/CD on job: ${env.JOB_NAME}
                Build Successed
                """
            )
        }
    }
}
