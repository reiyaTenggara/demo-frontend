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

        stage('kirim notifikasi ke slack') {
            steps {
                slackSend(message: "mulai job baru : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
        }

        stage("npm npman") {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }

        }

        stage("bucketd") {
            steps {
                s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'demo-frontend123', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: false, selectedRegion: 'ap-southeast-1', showDirectlyInBrowser: false, sourceFile: 'dist/**', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'FAILURE', profileName: 'demo-frontend123', userMetadata: []
            }

        }

    }

    post {
        aborted {
            slackSend(message: "build digagalkan secara manual : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            slackSend(message: "build failed : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        success {
            slackSend(message: "build success : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

    }
}