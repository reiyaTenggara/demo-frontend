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
        stage('Send notification to Slack') {
            steps {
                slackSend message: """\
                Starting CI/CD on job: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                <${env.BUILD_URL}console|View Output> || <${env.JOB_URL}|View Job> || <${env.JOB_DISPLAY_URL}/${env.BRANCH_NAME}| Open Blue Ocean>
                """
            }
        }

        stage('npm install & build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Upload dist folder to S3 bucket') {
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
            slackSend(color: "warning", message: "Build manually aborted:")
        }
        failure {
            slackSend(color: "danger", message: "Build failed:")
        }
        success {
            slackSend(color: "good", message: "Build succeeded:")
        }
    }
}