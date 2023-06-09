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
                def newUrl = env.BUILD_URL.replace("/job/", "/blue/organizations/jenkins/").replace("/" + env.BUILD_NUMBER + "/", "/detail/")
                slackSend(message: "Starting CI/CD on Repo/Branch: ${env.JOB_NAME}, Build Number: ${env.BUILD_NUMBER}, link: (<${newUrl}|Open>)")
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
            slackSend(color: "warning", message: "Build manually aborted: ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            slackSend(color: "red", message: "Build failed: ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend(color: "good", message: "Build succeeded: ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}