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
        stage("npm npman") {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }

        }

        stage("bucketd") {
            steps {
                sh 'cd dist'
                s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'demo-frontend123', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: true, selectedRegion: 'ap-southeast-1', showDirectlyInBrowser: false, sourceFile: '*/*', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'FAILURE', profileName: 'demo-frontend123', userMetadata: []
            }

        }

    }
}